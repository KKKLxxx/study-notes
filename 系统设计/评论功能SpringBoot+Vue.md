# 评论功能SpringBoot+Vue

评论功能主要是通过递归实现的，后端要根据从数据库中查询到的内容递归地生成一个树状的结构，前端要根据这个树状结构递归地生成评论组件

## 一、数据库设计

关键字段如下

| 字段名      | 类型    | 注释           |
| ----------- | ------- | -------------- |
| id          | int     | 评论id         |
| pid         | int     | 父评论id       |
| article_id  | int     | 所评论文章的id |
| user_id     | int     | 评论人id       |
| content     | varchar | 评论内容       |
| create_time | varchar | 评论时间       |

## 二、后端查询文章评论的过程

1、后端接收前端传过来的`article_id`，在数据库中查询出来所有`article_id`相符的评论

```java
@ResponseBody
@GetMapping(value = "{articleId}")
public HashMap<String, Object> getComment(@PathVariable(value = "articleId") int articleId) {
    HashMap<String, Object> result = new HashMap<>();
    List<Comment> comment = commentService.getCommentsByArticleId(articleId);
    result.put("data", comment);
    return result;
}

@Override
public List<Comment> getCommentsByArticleId(Integer articleId) {
	QueryWrapper<Comment> queryWrapper = new QueryWrapper<>();
    queryWrapper.lambda().eq(Comment::getArticleId, articleId);
    List<Comment> comments = commentMapper.selectList(queryWrapper);
    return TreeNodeUtil.assembleTree(comments);
}
```

2、利用TreeNodeUtil工具类封装为树状结构

```java
public static <T extends BaseTreeNode> List<T> assembleTree(List<T> listNodes) {
    List<T> newTreeNodes = new ArrayList<>();
    // 只添加第一层评论
    newTreeNodes.addAll(listNodes.stream().filter(t -> t.getPid() == null)
                .collect(Collectors.toList()));
    // 循环处理子节点数据
    for (T t : newTreeNodes) {
        assembleTree(t, listNodes);
    }
    return newTreeNodes;
}

// node为当前评论
// listNodes为待处理评论
static <T extends BaseTreeNode> void assembleTree(T node, List<T> listNodes) {
    if (node != null && !CollectionUtils.isEmpty(listNodes)) {
        // 只保留当前评论的回复，并通过addReplyComments将该回复添加到node的replyComments列表中
        listNodes.stream().filter(t -> Objects.equals(t.getPid(), node.getId()))
            .forEachOrdered(node::addReplyComments);
        // 递归处理回复的回复
        if (!CollectionUtils.isEmpty(node.getReplyComments())) {
        	for (Object t : node.getReplyComments()) {
                assembleTree((T) t, listNodes);
            }
        }
    }
}
```

​	其中，`Comment`类的结构与数据库中的结构相同,`T extends BaseTreeNode`的结构可以看作在`Comment`类的基础上添加了一个`List<BaseTreeNode> replyComments`字段用于保存每条评论的“回复”，这些“回复”也属于评论

​	在函数`public static <T extends BaseTreeNode> List<T> assembleTree(List<T> listNodes)`中，先筛选出来所有的一级评论。然后对每条一级评论，调用`static <T extends BaseTreeNode> void assembleTree(T node, List<T> listNodes)`，将其所有回复添加到`replyComments`字段中。由于评论是多级的，所以需要递归调用

```
// 只保留当前评论的回复，并通过addReplyComments将该回复添加到node的replyComments列表中
listNodes.stream().filter(t -> Objects.equals(t.getPid(), node.getId()))
        .forEachOrdered(node::addReplyComments);
```

这段代码的作用已经写在了注释中，其中`forEachOrdered`方法是为了保持评论原有的时间顺序，如果调用`forEach`可能乱序

## 三、前端展示评论

前端展示评论与后端`TreeNodeUtil`处理评论的思路类似，前端在评论功能上使用了两个组件`Comment.vue`与`Reply.vue`

```
<Comment>
	<评论编辑区/>
	<List>
		<ListItem v-for=所有一级评论>
			<评论详情/>
			<Reply v-if=存在回复内容 传入当前评论的所有回复/>
		</ListItem>
	</List>
</Comment>
```

```
<Reply>
	<List>
		<ListItem v-for=所有一级评论（相对的一级）>
			<评论详情/>
			<Reply v-if=存在回复内容 传入当前评论的所有回复/>
		</ListItem>
	</List>
</Reply>
```

两个组件之间的差异就在于评论编辑区，因为如果编辑区也递归显示的话，当级数太多时可能会错位

