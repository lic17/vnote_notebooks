# informer
Reflector收到事件通知将对应的 事件放入 -> Delta FIFO Queue 然后 SharedInformer 会不断从 Delta FIFO Queue 读取事件更新缓存
SharedInformer 通过 Workqueue 将数据同步给控制器 ，会将Update ，Create ，Delete 事件加入Workqueue 中，然后所有控制器排队读取， 通过注入的回调函数来执行相对应的操作，当操作失败，将事件重新放回队列，排队下次再试，成功就从 Workqueue删除该事件
informer->Reflector->