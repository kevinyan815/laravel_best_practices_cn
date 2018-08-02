

### 所有的业务要封装在Service类里

上面说了模型中应该只写与数据表抽象相关的代码，但是业务逻辑往往都是伴随着数据更新的，所以业务代码应该单独抽离到Service层级，由Service再去调用Model里定义的方法来实现数据更新。

```
class AritcleService
{
    public function setArticleOffline(Article $article)
    {
        if ($article->is_top == 1) {//如果是置顶文章需要将文章取消置顶
            $this->cancelTopArticle($article);
        }
        $article->status = Article::STATUS_OFFLINE;
        $article->offline_time = date('Y-m-d H:i:s');
        $article->save();

        return true;
    }

    /**
     * 取消文章的置顶
     * @param \App\Models\Article $article
     */
     public function cancelTopArticle($article)
     {
         if (TopArticle::specificArticle($article->id)->count()) {
             //删除置顶表里的记录(待上线的置顶文章上线后置顶表中才能有相应的记录)
             TopArticle::specificArticle($article->id)->delete();
         }
         //将文章的is_top字段置为0
         $article->is_top = 0;
         $article->save();

         return true;
     }
}
```

#### 单一业务中涉及多个Model的读写

```
class MemberService
{
    /**
     * 恢复会员卡状态和库存
     *
     * @param \App\Models\Card $card
     * @return boolean
     */
    public function restoreCard(Card $card)
    {
        if($card->status != Card::STATUS_PREUSED) {//只有发放中状态的卡才能恢复
            return false;
        }
        DB::beginTransaction();
        try {
            $card->status = Card::STATUS_UNUSED;//将这张卡号设置为未使用
            $card->save();
            RemainingNum::firstData()->increment('remaining_num');//将剩余会员卡数加1
            DB::commit();
        } catch (QueryException $exc) {
            \Log::error('Restore Card Error: ' . $exc->getMessage());
            DB::rollback();
            return false;
        }
        
        return true;
    }
}
```
