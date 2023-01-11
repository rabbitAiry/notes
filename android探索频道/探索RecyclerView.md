#### 探索ScrollToPosition是怎么工作的
- 负责滑动逻辑的是LinearLayoutManager
	- 滑动的距离是使得目标view可见的最小距离
```java
@Override  
public void scrollToPosition(int position) {  
    mPendingScrollPosition = position;  
    mPendingScrollPositionOffset = INVALID_OFFSET;  
    if (mPendingSavedState != null) {  
        mPendingSavedState.invalidateAnchor();  
    }  
    requestLayout();  
}
```

- requestLayout()的实现在View中
	- 该方法会在view内部发生变化需要重新绘制的时候
```java
public void requestLayout() {  
    if (mMeasureCache != null) mMeasureCache.clear();  
  
    if (mAttachInfo != null && mAttachInfo.mViewRequestingLayout == null) {  
        // Only trigger request-during-layout logic if this is the view requesting it,  
        // not the views in its parent hierarchy        ViewRootImpl viewRoot = getViewRootImpl();  
        if (viewRoot != null && viewRoot.isInLayout()) {  
            if (!viewRoot.requestLayoutDuringLayout(this)) {  
                return;  
            }  
        }  
        mAttachInfo.mViewRequestingLayout = this;  
    }  
  
    mPrivateFlags |= PFLAG_FORCE_LAYOUT;  
    mPrivateFlags |= PFLAG_INVALIDATED;  
  
    if (mParent != null && !mParent.isLayoutRequested()) {  
        mParent.requestLayout();  
    }  
    if (mAttachInfo != null && mAttachInfo.mViewRequestingLayout == this) {  
        mAttachInfo.mViewRequestingLayout = null;  
    }  
}
```

- 结论：触发滑动至目标值的条件是`mPendingScrollPosition = position;` ，这个参数能在onLayoutChildren()见到

#### 探索smoothScrollToPosition
- 调用的实际是LinearLayoutManager的实例
```java
public void smoothScrollToPosition(RecyclerView recyclerView, RecyclerView.State state,  
        int position) {  
    LinearSmoothScroller linearSmoothScroller =  
            new LinearSmoothScroller(recyclerView.getContext());  
    linearSmoothScroller.setTargetPosition(position);  
    startSmoothScroll(linearSmoothScroller);  
}
```

- 父类LayoutManager提供了startSmoothScroll()的实现
```java
public void startSmoothScroll(SmoothScroller smoothScroller) {  
	// 停下先前动画
    if (mSmoothScroller != null && smoothScroller != mSmoothScroller  
            && mSmoothScroller.isRunning()) {  
        mSmoothScroller.stop();  
    }  
    // 开始当前动画
    mSmoothScroller = smoothScroller;  
    mSmoothScroller.start(mRecyclerView, this);  
}
```

- 父类SmoothScroller提供了start()的实现
	- 该方法将动画分成许多步骤，在每一步中，检查是否找到了target view
```java
void start(RecyclerView recyclerView, LayoutManager layoutManager) {  
  
    // Stop any previous ViewFlinger animations now because we are about to start a new one.  
    recyclerView.mViewFlinger.stop();  
  
    // ...
  
    mRecyclerView = recyclerView;  
    mLayoutManager = layoutManager;  
    if (mTargetPosition == RecyclerView.NO_POSITION) {  
        throw new IllegalArgumentException("Invalid target position");  
    }  
    mRecyclerView.mState.mTargetPosition = mTargetPosition;  
    mRunning = true;  
    mPendingInitialRun = true;  
    mTargetView = findViewByPosition(getTargetPosition());  
    onStart();  
    mRecyclerView.mViewFlinger.postOnAnimation();  
  
    mStarted = true;  
}
```

- mRecyclerView.mViewFlinger.postOnAnimation()
- mRecyclerView.mViewFlinger.run()
	- flinger是其内部实现
```java
public void run() {  
    if (mLayout == null) {  
        stop();  
        return; // no layout, cannot scroll.  
    }  
  
    mReSchedulePostAnimationCallback = false;  
    mEatRunOnAnimationRequest = true;  
  
    consumePendingUpdateOperations();  
  
    // Keep a local reference so that if it is changed during onAnimation method, it won't    // cause unexpected behaviors    final OverScroller scroller = mOverScroller;  
    if (scroller.computeScrollOffset()) {  
        final int x = scroller.getCurrX();  
        final int y = scroller.getCurrY();  
        int unconsumedX = x - mLastFlingX;  
        int unconsumedY = y - mLastFlingY;  
        mLastFlingX = x;  
        mLastFlingY = y;  
        int consumedX = 0;  
        int consumedY = 0;  
  
        // Nested Pre Scroll  
        mReusableIntPair[0] = 0;  
        mReusableIntPair[1] = 0;  
        if (dispatchNestedPreScroll(unconsumedX, unconsumedY, mReusableIntPair, null,  
                TYPE_NON_TOUCH)) {  
            unconsumedX -= mReusableIntPair[0];  
            unconsumedY -= mReusableIntPair[1];  
        }  
  
        // Based on movement, we may want to trigger the hiding of existing over scroll  
        // glows.        if (getOverScrollMode() != View.OVER_SCROLL_NEVER) {  
            considerReleasingGlowsOnScroll(unconsumedX, unconsumedY);  
        }  
  
        // Local Scroll  
        if (mAdapter != null) {  
            mReusableIntPair[0] = 0;  
            mReusableIntPair[1] = 0;  
            scrollStep(unconsumedX, unconsumedY, mReusableIntPair);  
            consumedX = mReusableIntPair[0];  
            consumedY = mReusableIntPair[1];  
            unconsumedX -= consumedX;  
            unconsumedY -= consumedY;  
  
            // If SmoothScroller exists, this ViewFlinger was started by it, so we must  
            // report back to SmoothScroller.            SmoothScroller smoothScroller = mLayout.mSmoothScroller;  
            if (smoothScroller != null && !smoothScroller.isPendingInitialRun()  
                    && smoothScroller.isRunning()) {  
                final int adapterSize = mState.getItemCount();  
                if (adapterSize == 0) {  
                    smoothScroller.stop();  
                } else if (smoothScroller.getTargetPosition() >= adapterSize) {  
                    smoothScroller.setTargetPosition(adapterSize - 1);  
                    smoothScroller.onAnimation(consumedX, consumedY);  
                } else {  
                    smoothScroller.onAnimation(consumedX, consumedY);  
                }  
            }  
        }  
  
        if (!mItemDecorations.isEmpty()) {  
            invalidate();  
        }  
  
        // Nested Post Scroll  
        mReusableIntPair[0] = 0;  
        mReusableIntPair[1] = 0;  
        dispatchNestedScroll(consumedX, consumedY, unconsumedX, unconsumedY, null,  
                TYPE_NON_TOUCH, mReusableIntPair);  
        unconsumedX -= mReusableIntPair[0];  
        unconsumedY -= mReusableIntPair[1];  
  
        if (consumedX != 0 || consumedY != 0) {  
            dispatchOnScrolled(consumedX, consumedY);  
        }  
  
        if (!awakenScrollBars()) {  
            invalidate();  
        }  
  
        // We are done scrolling if scroller is finished, or for both the x and y dimension,  
        // we are done scrolling or we can't scroll further (we know we can't scroll further        // when we have unconsumed scroll distance).  It's possible that we don't need        // to also check for scroller.isFinished() at all, but no harm in doing so in case        // of old bugs in Overscroller.        boolean scrollerFinishedX = scroller.getCurrX() == scroller.getFinalX();  
        boolean scrollerFinishedY = scroller.getCurrY() == scroller.getFinalY();  
        final boolean doneScrolling = scroller.isFinished()  
                || ((scrollerFinishedX || unconsumedX != 0)  
                && (scrollerFinishedY || unconsumedY != 0));  
  
        // Get the current smoothScroller. It may have changed by this point and we need to  
        // make sure we don't stop scrolling if it has changed and it's pending an initial        // run.        SmoothScroller smoothScroller = mLayout.mSmoothScroller;  
        boolean smoothScrollerPending =  
                smoothScroller != null && smoothScroller.isPendingInitialRun();  
  
        if (!smoothScrollerPending && doneScrolling) {  
            // If we are done scrolling and the layout's SmoothScroller is not pending,  
            // do the things we do at the end of a scroll and don't postOnAnimation.  
            if (getOverScrollMode() != View.OVER_SCROLL_NEVER) {  
                final int vel = (int) scroller.getCurrVelocity();  
                int velX = unconsumedX < 0 ? -vel : unconsumedX > 0 ? vel : 0;  
                int velY = unconsumedY < 0 ? -vel : unconsumedY > 0 ? vel : 0;  
                absorbGlows(velX, velY);  
            }  
  
            if (ALLOW_THREAD_GAP_WORK) {  
                mPrefetchRegistry.clearPrefetchPositions();  
            }  
        } else {  
            // Otherwise continue the scroll.  
  
            postOnAnimation();  
            if (mGapWorker != null) {  
                mGapWorker.postFromTraversal(RecyclerView.this, consumedX, consumedY);  
            }  
        }  
    }  
  
    SmoothScroller smoothScroller = mLayout.mSmoothScroller;  
    // call this after the onAnimation is complete not to have inconsistent callbacks etc.  
    if (smoothScroller != null && smoothScroller.isPendingInitialRun()) {  
        smoothScroller.onAnimation(0, 0);  
    }  
  
    mEatRunOnAnimationRequest = false;  
    if (mReSchedulePostAnimationCallback) {  
        internalPostOnAnimation();  
    } else {  
        setScrollState(SCROLL_STATE_IDLE);  
        stopNestedScroll(TYPE_NON_TOUCH);  
    }  
}
```

- 没辙了


