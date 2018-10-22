- handler机制实现的礼物连送，实现如下

  ```java
  private void pumpGift() {
      //make sure run main thread
      if (mPumping) {
          return;
      } else {
          mPumping = true;
      }
      if (mList.size() == 0 || mTrackView == null) {
          mPumping = false;
          return;
      }
      if (mCurDisplayNum < GiftTrackView.TRACK_NUM && mList.size() > 0) {
          int availableGiftTrackNo = mTrackView.getAvailableGiftTrack();
          while (mList.size() > 0 && availableGiftTrackNo != -1) {
              //说明有数据，也有空置的轨道
              LiveSendGiftNtBean giftItem = generateValidGiftItem();
              if (giftItem != null) {
                  mCurDisplayNum++;
                  //...省略
              } else {
                  //...省略
                  break;
              }
          }
          if (availableGiftTrackNo != -1) {
              //...省略
          }
      }
      invoke(new Runnable() {
          @Override
          public void run() {
              pumpGift();
          }
      }, 500);
      mPumping = false;
  }
  
  private void invoke(Runnable runnable, long delay) {
      if (null == mHandler) {
          mHandler = new Handler(Looper.getMainLooper());
      }
      mHandler.postDelayed(runnable, delay);
  }
  ```

  从而执行每500毫秒检测一次轨道的任务，在视图要结束的时候，通过`removeMessage`来结束循环调用。

  ```java
  public void cancelScheduleThread() {
      if (null != mHandler) {
          mHandler.removeCallbacksAndMessages(null);
          mHandler = null;
      }
      //...
  }
  ```

- 项目中有一个预热列表需要加入倒计时，且计时器跑完移除对应的项目。如下图：

  ![]("./src/tk.png")

  网上看到一篇写recyclerView计时器的，它的每个item都需要计时，所以都开了`CountDownTimer`，并且在ViewHolder的`onViewDetachedFromWindow`未做任何处理。随着列表的滑动，卡顿不言而喻。我采用了订阅者模式解决这个问题，核心代码如下：

  ```java
  public final class TimeEventEmitter extends Observable {
  
      private TimerTask mTimerTask;
  
      private static final int INTERVAL = 1000 * 10;//10s 一次
  
      //只要创建大于0 就会执行定时刷新
      private int mInterval;
  
      //。。。
  
      public TimeEventEmitter(int interval) {
          mInterval = interval;
      }
  
      public static TimeEventEmitter emit(int interval) {
          TimeEventEmitter eventEmitter = new TimeEventEmitter(interval);
          eventEmitter.startTimerTask();
          return eventEmitter;
      }
  
      public final void startTimerTask() {
          if (mTimerTask == null) {
              mTimerTask = new EmitterSchedule();
          } else {
              //...
              return;
          }
          try {
              TaskEngine.getInstance().schedule(mTimerTask, mInterval > 0 ? mInterval : INTERVAL,
                      mInterval > 0 ? mInterval : INTERVAL);
          } catch (IllegalStateException ex) {
              //。。。
              cancelEmit();
              startTimerTask();
          }
      }
      //。。。
  
      private class EmitterSchedule extends TimerTask {
  
          @Override
          public void run() {
              //work thread
              setChanged();
              //send a heart beat
              notifyObservers();
          }
      }
  }
  ```

  在需要订阅的视图处，订阅时间事件：

  ```java
  @Override
  protected void onViewAttachedToWindow(@NonNull InteractTaskHolder holder) {
      TaskDataManager.receive(holder);
  }
  
  @Override
  protected void onViewDetachedFromWindow(@NonNull InteractTaskHolder holder) {
      TaskDataManager.reject(holder);
  }
  
  public static class InteractTaskHolder extends RecyclerView.ViewHolder implements Observer {
  
      //private members
  
      public InteractTaskHolder(View itemView) {
          //init...
      }
  
      public void bindHolderWithData(final InteractListBean interactListBean, @Nullable final int flag) {
          //...
          long remain = interactListBean.time - current;
          //...
          if (interactListBean.sid == 0) {
              //可领取奖励...
          } else {
              //...
              if (remain < 0) {
                  //已经结束预热
                  TaskDataManager.reject(this);
                  //...
                  return;
              }
          }
      }
  
      @Override
      public void update(Observable o, Object arg) {
          if (null == mData || null == handler) {
              return;
          }
          //...
          final long remain = mData.time - current;
          if (remain < 0) {
              TaskDataManager.reject(this);
              //...
              return;
          }
          handler.post(new Runnable() {
              @Override
              public void run() {
                  tvTaskTimer.setText(DateUtil.formatMinsnSecsChs(itemView.getContext().getString(R.string.time_remain),
                          (int) remain, false));
              }
          });
      }
  }
  ```

  同时在`onViewDetachedFromWindow`中解除订阅。

- 
