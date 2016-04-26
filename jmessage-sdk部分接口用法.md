####jmessage-sdk部分接口的介绍和使用

- 会话Conversation

会话在JChat中是聊天的载体，发消息需要先用会话创建消息，收消息sdk会将消息放入相应的会话，如果本地没有会话，则会新建一个会话，并将之加到会话，所以上层只需要刷新会话列表。以下是得到会话的两种方式：

得到会话列表：
```
    //此方法得到本地保存的历史会话列表
    List<Conversation> mDatas = JMessageClient.getConversationList();
```

创建会话：
```
    //创建单聊
    Conversation conv = Conversation.createSingleConversation(username);
    //创建群聊
    Conversation conv = Conversation.createGroupConversation(groupId);
```

使用Conversation得到TargetInfo，如果该Conversation为单聊，则得到UserInfo，如果为群聊则得到GroupInfo：
```
    UserInfo userInfo = (UserInfo) conv.getTargetInfo();
    GroupInfo groupInfo = (GroupInfo) conv.getTargetInfo();
```

使用Conversation得到历史消息：
```
    //得到所有消息
    List<Message> msgList = conv.getAllMessage();
    //按时间顺序（最先收到的消息放在最前），从start开始得到offset条消息
    List<Message> msgList = conv.getMessagesFromOldest(start, offset);
    //按照最近消息顺序（最近的消息放在最前），从start开始得到offset条消息
    List<Message> msgList = conv.getMessagesFromNewest(start, offset);
    //根据messageId得到某一条消息
    Message msg = conv.getMessage(messageId);
    //根据messageId删除某一条消息
    conv.deleteMessage(messageId);
```

- 接收消息

 在Activity的onCreate()方法中先调用

```
 JMessageClient.registerEventReceiver(this);
```
 注意：如果你是用了jmessage－sdk提供的Notification，在进入了聊天界面后，需要在onCreate()或者onResume()方法中调用
 ```
 JMessageClient.enterSingleConversaion(targetId);
 //或者JMessageClient.enterGroupConversaion(groupId);
 ```
 并且在onPause()中调用
 ```
 JMessageClient.exitConversaion();
 ```
 这样就可以控制Notification是否要显示给用户。
 接下来你可以重写onEvent()方法来接收消息，并刷新聊天界面，如下所示：

> demo ChatActivity.java onEvent()

```
    /**
     * 接收消息类事件
     *
     * @param event 消息事件
     */
    public void onEvent(MessageEvent event) {
        final Message msg = event.getMessage();
        
        //可以在这里创建Notification
        ...
        
        //刷新消息
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                //收到消息的类型为单聊
                if (msg.getTargetType() == ConversationType.single) {
                    String targetID = ((UserInfo) msg.getTargetInfo()).getUserName();
                    //判断消息是否在当前会话中
                    if (targetID.equals(mTargetId)) {
                        Message lastMsg = mChatAdapter.getLastMsg();
                        //收到的消息和Adapter中最后一条消息比较，如果最后一条为空或者不相同，则加入到MsgList
                        if (lastMsg == null || msg.getId() != lastMsg.getId()) {
                            mChatAdapter.addMsgToList(msg);
                        } else {
                            mChatAdapter.notifyDataSetChanged();
                        }
                    }
                } else {
                    long groupId = ((GroupInfo)msg.getTargetInfo()).getGroupID();
                    if (!mIsSingle && groupId == mGroupId) {
                        Message lastMsg = mChatAdapter.getLastMsg();
                        if (lastMsg == null || msg.getId() != lastMsg.getId()) {
                            mChatAdapter.addMsgToList(msg);
                        } else {
                            mChatAdapter.notifyDataSetChanged();
                        }
                    }
                }
            }
        });
    }
```

其中，MessageEvent中的Message类型可以是普通消息，也可以是EventNotification或者Custom类型，可以根据ContentType分别处理。
消息的状态：消息状态可以使用msg.getStatus()获得，为Enum类型，在getView时可以根据消息的状态做相应的展示。
```
    switch (msg.getStatus) {
        case send_success:
        case send_fail:
        case send_going:
        case receive_success:
        case receive_fail:
        case receive_going:
    }
```

- 发送消息

  发送文本消息：
```
   //其中msgContent为string，mConv为Conversation
   TextContent content = new TextContent(msgContent);
   Message msg = mConv.createSendMessage(content);
   JMessageClient.sendMessage(msg);
   //发送时要注册一个callback，监听消息是否发送成功
   if (!msg.isSendCompleteCallbackExists()) {
            msg.setOnSendCompleteCallback(new BasicCallback() {
                @Override
                public void gotResult(final int status, final String desc) {
                   if (status == 0) {
                   //发送成功
                   } 
                }
            }
    });
```

  发送语音消息：
```
  //mRecAudioFile为录音文件，duration为录音时长
  VoiceContent content = new VoiceContent(myRecAudioFile, duration);
  Message msg = mConv.createSendMessage(content);
  JMessageClient.sendMessage(msg);
  //发送时要注册一个callback，监听消息是否发送成功
   if (!msg.isSendCompleteCallbackExists()) {
            msg.setOnSendCompleteCallback(new BasicCallback() {
                @Override
                public void gotResult(final int status, final String desc) {
                   if (status == 0) {
                   //发送成功
                   } 
                }
            }
    });
```

  发送图片消息
```
  ImageContent.createImageContentAsync(bitmap, new ImageContent.CreateImageContentCallback() {
      @Override
      public void gotResult(int status, String desc, ImageContent imageContent) {
          if (status == 0) {
              Message msg = mConv.createSendMessage(imageContent);
              JMessageClient.sendMessage(msg);
          }
      }
  });
  //如果图片正在发送，可以注册callback获取进度
        if (!msg.isContentUploadProgressCallbackExists()) {
            msg.setOnContentUploadProgressCallback(new ProgressUpdateCallback() {
                @Override
                public void onProgressUpdate(double v) {
                    String progressStr = (int) (v * 100) + "%";
                    Log.d(TAG, "msg.getId: " + msg.getId() + " progress: " + progressStr);
                    holder.progressTv.setText(progressStr);
                }
            });
        }
        if (!msg.isSendCompleteCallbackExists()) {
            msg.setOnSendCompleteCallback(new BasicCallback() {
                @Override
                public void gotResult(final int status, String desc) {
                   
                }
            });

```

- UserInfo相关接口用法：

得到UserInfo：
```
JMessageClient.getUserInfo(targetId, new GetUserInfoCallback() {
   @Override
   public void gotResult(final int status, String desc, final UserInfo userInfo) {
      if (status == 0) {
        //do something
      } else {
        //get userInfo failed
      }      
   }
 }
```

得到用户头像(分为拿大头像和小头像两个接口)
```
    //拿头像之前先判断userInfo是否为空，并且要判断该用户是否设置了头像
    if (userInfo != null && !TextUtils.isEmpty(userInfo.getAvatar())) {
        //异步接口，拿到头像后，sdk会缓存
        userInfo.getAvatarBitmap(new GetAvatarBitmapCallback() {
            @Override
            public void gotResult(int status, String desc, Bitmap bitmap) {
                if (status == 0) {
                    holder.headIcon.setImageBitmap(bitmap);
                } else {
                    holder.headIcon.setImageResource(R.drawable.head_icon);
                    HandleResponseCode.onHandle(mContext, status, false);
                }
            }
        });
    } else {
        holder.headIcon.setImageResource(R.drawable.head_icon);
    }
    
    //得到大头像
    if (userInfo != null && !TextUtils.isEmpty(userInfo.getAvatar)) {
        //拿大头像sdk不会缓存，如有需要可以自己添加缓存
        userInfo.getBigAvatarBitmap(new GetAvatarBitmapCallback() {
            @Override
            public void gotResult(int status, String desc, Bitmap bitmap) {
                //TODO 
            }
        });
    }
```

用UserInfo判断是否加入了黑名单：
```
    //等于1表示加入了黑名单
    if (1 == userInfo.getBlacklist()) {}
```

用UserInfo或者GroupInfo设置免打扰：
```
        //设置免打扰,1为将当前用户或群聊设为免打扰,0为移除免打扰
        if (mIsGroup) {
            mGroupInfo.setNoDisturb(checked ? 1 : 0, new BasicCallback() {
                @Override
                public void gotResult(int status, String desc) {
                    dialog.dismiss();
                    if (status == 0) {
                        if (checked) {
                            Toast.makeText(mContext, mContext.getString(R.string.set_do_not_disturb_success_hint),
                                    Toast.LENGTH_SHORT).show();
                        } else {
                            Toast.makeText(mContext, mContext.getString(R.string.remove_from_no_disturb_list_hint),
                                    Toast.LENGTH_SHORT).show();
                        }
                    //设置失败,恢复为原来的状态
                    } else {
                        if (checked) {
                            mChatDetailView.setNoDisturbChecked(false);
                        } else {
                            mChatDetailView.setNoDisturbChecked(true);
                        }
                        HandleResponseCode.onHandle(mContext, status, false);
                    }
                }
            });
        } else {
            mUserInfo.setNoDisturb(checked ? 1 : 0, new BasicCallback() {
                @Override
                public void gotResult(int status, String desc) {
                    dialog.dismiss();
                    if (status == 0) {
                        if (checked) {
                            Toast.makeText(mContext, mContext.getString(R.string.set_do_not_disturb_success_hint),
                                    Toast.LENGTH_SHORT).show();
                        } else {
                            Toast.makeText(mContext, mContext.getString(R.string.remove_from_no_disturb_list_hint),
                                    Toast.LENGTH_SHORT).show();
                        }
                        //设置失败,恢复为原来的状态
                    } else {
                        if (checked) {
                            mChatDetailView.setNoDisturbChecked(false);
                        } else {
                            mChatDetailView.setNoDisturbChecked(true);
                        }
                        HandleResponseCode.onHandle(mContext, status, false);
                    }
                }
            });
        }
```

另外可以从userInfo得到username、nickname等属性，此处不再一一赘述。

- GroupInfo相关接口用法：
```
    //得到群主用户名
    String groupOwnerId = groupInfo.getGroupOwner();
    //得到群成员信息
    List<UserInfo> membersInfo = groupInfo.getGroupMembers();
    String groupName  = groupInfo.getGroupName();
```

- 创建群聊
```
    JMessageClient.createGroup("", "", new CreateGroupCallback() {
        @Override
        public void gotResult(int status, final String desc, final long groupId) {
        }
    });
```
