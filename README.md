# vue-pull-to-refresh-and-load-more
a scroll component with function of pull down refresh and pull up load more for vue

I create a scroller component for vue base iscroll-lite. it support more than one scroller in one page

```vue
<template>
   <div>
      <scroller :hswipe-stop-scroll="true" :init-load-more="true" :wrapper-id="'wrapper0'" :scroll-id="'scroll0'" :need-refresh="true" :refresh-call="refresh" :load-more-call="loadMore" :has-more-data="alls[0].hasMore" :more-tip="alls[0].list.length>0">
     </scroller>
   </div>
   <div>
      <scroller :hswipe-stop-scroll="true" :wrapper-id="'wrapper1'" :scroll-id="'scroll1'" :need-refresh="true" :refresh-call="refresh" :load-more-call="loadMore" :has-more-data="alls[1].hasMore" :more-tip="alls[1].list.length>0">      
      </scroller>
   </div>
</template>
<script>
import Scroller from './iscroller/ZZScroll.vue'

const PageSize = 20;

export default{
    components: { Scroller},

    data () {
      return {
        nowType: 0,
        alls: [
          { list: [], pageNo: 1, hasMore: true },
          { list: [], pageNo: 1, hasMore: true }
        ]
        
    },

    methods: {
      initList (type, isRefresh, succCall, errCall) {
        succCall = succCall || function(){};
        errCall = errCall || function(){};
        let opt = {}, self = this, temp = this.alls[type];
        if(isRefresh) {
          temp.list.length = 0;   //不使用 =[]; 会出现短暂的空白
          temp.pageNo = 1;
          temp.hasMore = true;
        }
        opt.pageNo = temp.pageNo;
        opt.pageSize = PageSize;
        opt.status = type;
        Http.post('SAAS_H5_GetItemOrderList', opt).then((data) => {
          data = data.body;
          temp.list = temp.list.concat(data.orderList);
          self.storePhone = data.storePhone;        //保存店铺电话
          if( temp.pageNo >= Math.ceil(data.totalCount / PageSize) ){   //判断数据是否加载完全
            temp.hasMore = false;
            succCall(data);
            return;
          }
          temp.pageNo++;
          succCall(data);
        }, () => {
          errCall();
        });
      },

      refresh (scrollId) {
        let self = this;
        let type=scrollId.slice(-1);
        this.initList(type, true, function(){
          self.$broadcast('refresh-finish', {errorCode: 0, scrollId: scrollId});
        }, function(){
          self.$broadcast('refresh-finish', {errorCode: -1, scrollId: scrollId});
        })
      },
      loadMore (scrollId) {
        let self = this;
        let type=scrollId.slice(-1);
        this.initList(type, false, function(){
          self.$broadcast('load-finish', {errorCode: 0, scrollId: scrollId});
        }, function(){
          self.$broadcast('load-finish', {errorCode: -1, scrollId: scrollId});
        })
      }
    }
}
</script>
```

as you can see, this scroller has some props:
- hswipe-stop-scroll:  work for when you swipe scroll, stop vertical scroll.
- init-load-more:   work for if you need load data when init this component.
- wrapper-id and scroll-id:  work for specify many scroller in one page.
- need-refresh:  work for if you need the refresh and load more function.
- refresh-call:  work for pull down refresh call.
- load-more-call:　work for pull up load more call.
- has-more-data:　work for load more status, if false, it will show 'no more data'.
- more-tip: work for control 'no more data' tip shows.

above, I give a example to use in a ajax request base vue. when you $broadcast 'refresh-finish' or 'load-finish'. remeber the transfer scrollId. and errorCode:0 is success. errorCode:-1 is false.

![](http://ww3.sinaimg.cn/mw690/a5e2541bgw1f8qrr6en35j20k00zkdin.jpg)
![](http://ww3.sinaimg.cn/mw690/a5e2541bgw1f8qrr5znlsj20k00zkgpx.jpg)
![](http://ww1.sinaimg.cn/mw690/a5e2541bgw1f8qrr5mh9wj20k00zktcd.jpg)
![](http://ww4.sinaimg.cn/mw690/a5e2541bgw1f8qrr54yfjj20k00zk42c.jpg)

