<!-- 模块大标题 -->
# PC直播模块功能说明
<!-- 模块说明 -->
包含了pc直播逻辑描述以及pc和客户端进行数据交互的通信协议等


<!-- 页面bridge交互说明 -->
## 连麦数据交互协议说明


#### 通知所有客户端报告自己的在线状态（老师发起）
- pc刷新页面，获取到"getListPersonInfoInLive"列表的数据后，开始重新让客户端立马发送一次在线消息，
- 3s后，判断各种用户的在线状态，如果没有上麦的话，判断连麦列表用户，1个就弹窗，多个就不弹窗
```
ext：refresh
data：{"userId":"21","roleName":"教师","onlineNum":12312,"applyNum":12,"come":1,mainStream:{audio:true,video:false}}
说明：
  1）【老师】告知所有端，广播自己的在线信息；
  2）onlineNum：主讲老师告知所有学生端，在线人数（可能不传，客户端做不传的判断）
  3）applyNum：主讲老师告知所有学生端，申请连麦人数（可能不传，客户端做不传的判断）
  4）老师端：come == 1的时候，所有端要立马发送自己的在线消息；
  5）come == 1的时候，老师立马发送本地音视频开关状态mainStream:{audio:true,video:false}
```
```
ext：refresh
data：{"userId":"21","roleName":"学生","tellInfo":{"tellStatus":"living","tellType":"audio"},support:'tellphone'}
说明：
  1）【学生】告知所有端，广播自己的在线信息；
  2）老师端：come == 1的时候，所有端要立马发送自己的在线消息；
  3）老师端：come == 1的时候，学生端返回的数据包含连麦的情况
      a）连麦中：{"userId":"21","roleName":"学生",support:'tellphone',"tellInfo":{"tellStatus":"living","tellType":"audio"}}
      b）申请中：{"userId":"21","roleName":"学生",support:'tellphone',"tellInfo":{"tellStatus":"applying","tellType":"audio"}}
```

#### 学生进入直播间，发送一条进入的消息状态（学生发起）
- 学生进入直播间，发起一条newuser消息，老师接受到后，判断学生是否在连麦中；
注：在连麦中的话，给学生发送一条强制上麦的消息
注：主讲老师直接发送一条refresh消息，告知整体情况
```
ext：newuser
data：""
说明：告知老师端，我进来了
```

#### 老师强制学生上麦的消息（老师发起）
- 不管学生有没有准备好，强制某个学生用户上麦（音频、音视频）
- 场景：学生连麦中，突然中断了/退出了房间/关闭了软件，再次进来的时候，老师会通过此信息告知客户端，直接上麦
- 【暂时先不用这个方法，退出了房间，再进来就下麦了。】
```
ext：forceLive
data：{"userId":"21","tellType":"audio"}
说明：
    1）强制音视频：tellType = audio"
    2）强制音频： tellType = video
```

#### 老师强制学生下麦的消息（老师发起）
- 不管学生有没有准备好，强制某个学生用户下麦
- 场景：学生连麦中，老师想让学生下麦了，可以通过一个按钮直接让学生下麦
```
ext：forceFinishLive
data：{"userId":"21"}
```

#### 老师建议客户端发起连麦请求（老师发起）
```
ext：tell_tea_suggest_call
data：{"userId":"21"}
说明：让客户端主动发起连麦，但是客户端收到后会弹出建议弹窗
```

#### 老师撤销建议客户端发起连麦请求（老师发起）
```
ext：tell_tea_cancel_suggest_call
data：{"userId":"21"}
说明：取消之前发起的建议连麦的申请，让客户端连麦请求失效
```


#### 学生拒绝老师的邀请连麦请求（学生发起）
```
ext：tell_stu_refuse_suggest_call
data：{"userId":"21"}
说明：老师发来上麦邀请后，学生端弹出弹窗，用户点击拒绝后发起的一个信号
```

#### 学生申请连麦的请求（学生发起）
```
ext：tell_stu_apply_call
data：{"userId":"21","tellType":"audio"}
说明：
    1）强制音视频：tellType = audio"
    2）强制音频： tellType = video
```

#### 学生取消申请连麦的请求（学生发起）
```
ext：tell_stu_cancel_apply_call
data：{"userId":"21"}
说明：30s后tell_stu_apply_call会自动失效，所以这里取消有时效性哦
```

#### 老师拒绝学生申请连麦的请求（老师发起）
```
ext：tell_tea_refuse_apply_call
data：{"userId":"21"}
说明：老师拒绝后，客户端直接弹出拒绝结果，让学生死了这条心
```

#### 老师同意学生申请连麦的请求（老师发起）
```
ext：tell_tea_agree_apply_call
data：{"userId":"21","tellType":"audio"}
说明：老师拒绝后，客户端直接弹出拒绝结果，让学生死了这条心，tellType会带着，客户端可以不用
```

#### 学生主动问某个上麦的学生的个人信息（学生发起）
```
ext：tell_stu_ask_user_info
data：{"askUserId":"21"}
说明：学生进入直播课之后，由于没有正在上麦的学生信息，所以主动发起问一下谁在直播
```

#### 学生主动将自己的个人信息广播出去（学生发起）
```
ext：tell_stu_reply_user_info
data：{"userId":"","name":"","headUrl":""}
说明：
    1、学生上麦后，主动将自己的信息广播给所有人
    2、被其他学生问候，发现问的事自己，主动将自己的信息广播出去
```

#### 老师讲学生的连麦申请状态同步给其他老师（老师发起）
```
ext：tell_tea_sync_stu_status
data：{"userId":"",statusKey:"applyTime",statusValue:''}
说明：
   
```

#### 主讲老师发起的，告诉所有人清空老师和助教的聊天记录（老师发起）
```
ext：tell_all_delete_teacher_message
data：''   
```

#### 服务器端发起的，告诉所有人当前在线人数
```
ext：adminBroadcast
data：{"type": "groupOnlineCount","count": 123}   
```

#### 【新增】服务器端发起的，告知所有人弹框

```
ext：adminBroadcast
data {"type": "welfarePopUp", "userId": "", "info": {}}

```
userId为空字符串，在线的所有人都弹出弹框；
非空字符串时，含义为im中的userId，特定用户弹框

#### 【新增】学生端（app和web学生端）发起，告知服务端关闭了福利弹框

```
ext：tell_manager_closeWalfarePop
data：{"askUserId":""}
说明：学生点击关闭弹框时，在im中发送该消息
```

#### 【新增】服务器端发起的，告知所有人视频流和白板流地址

```
ext：adminBroadcast
data {"type": "trtcUrlNtf", "userId": "", "info": { "trtcUrl": "", "writeUrl": "" }}
说明：trtcUrl是视频流地址，writeUrl是白板流地址
```
