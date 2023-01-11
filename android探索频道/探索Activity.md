### 横屏导致app出bug的探索
- 使用`requestedOrientation = ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE`切换横屏
```
22:13:53.583  D  onCreate: 
22:13:53.592  D  onStart: 
22:13:53.597  D  onResume: 
22:13:53.639  D  onPause: 
22:13:53.648  D  onStop: 
22:14:17.868  D  onDestroy: 
22:14:17.961  D  onCreate: 
22:14:17.964  D  onStart: 
22:14:17.965  D  onResume: 
```
- 额外在manifest中使用`android:configChanges="orientation"`
```
22:14:17.961  D  onCreate: 
22:14:17.964  D  onStart: 
22:14:17.965  D  onResume: 
```



- 草稿
```
if (mLayout != null) {  
    checkForRelayout();  
}  
  
sendOnTextChanged(text, 0, oldlen, textLength);  
onTextChanged(text, 0, oldlen, textLength);
```