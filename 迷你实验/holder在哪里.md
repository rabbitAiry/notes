​	因为在发现待办事项app的RecyclerView中，理应点击完成按钮就会把当前选中项的holder删除，但是开发过程中发现了奇怪的现象，比如明明点击删除A项，在RecyclerView中却是B项消失了；又比如关闭app后重新打开会发现没有删掉。

​	查看了以前正确使用的版本，发现position的获取需要通过`holder.getAdapterPosition()`。所以做了一个对比，随意添加了十项内容并随机删除。当然，以下代码是能正常使用的代码

```java
holder.itemButtonDone.setOnClickListener(v -> {
    try {
        Log.d("NowAdapter", "position:"+position+"\t holder's position:"+holder.getAdapterPosition());
        listener.onButtonDoneClicked(item);
        itemList.remove(item);
        notifyItemRemoved(holder.getAdapterPosition());
    }catch (IndexOutOfBoundsException e){
        e.printStackTrace();
    }
});
```

