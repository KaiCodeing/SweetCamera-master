# SweetCamera
本工程主要是为了解决android各种机型的相机自动对焦问题，这里采用了一种兼容性比较好的方案，通过传感器进行对焦，希望能够帮助你解决自动对焦问题。

[博客详情解读][1]

预览图：

![效果图][2]

具体原理如图：

 ![enter description here][3]

核心对焦代码
```java
 if (event.sensor.getType() == Sensor.TYPE_ACCELEROMETER) {
            int x = (int) event.values[0];
            int y = (int) event.values[1];
            int z = (int) event.values[2];
            mCalendar = Calendar.getInstance();
            long stamp = mCalendar.getTimeInMillis();// 1393844912

            int second = mCalendar.get(Calendar.SECOND);// 53

            if (STATUE != STATUS_NONE) {
                int px = Math.abs(mX - x);
                int py = Math.abs(mY - y);
                int pz = Math.abs(mZ - z);
//                Log.d(TAG, "pX:" + px + "  pY:" + py + "  pZ:" + pz + "    stamp:"
//                        + stamp + "  second:" + second);
                double value = Math.sqrt(px * px + py * py + pz * pz);
                if (value > 1.4) {
//                    textviewF.setText("检测手机在移动..");
//                    Log.i(TAG,"mobile moving");
                    STATUE = STATUS_MOVE;
                } else {
//                    textviewF.setText("检测手机静止..");
//                    Log.i(TAG,"mobile static");
                    //上一次状态是move，记录静态时间点
                    if (STATUE == STATUS_MOVE) {
                        lastStaticStamp = stamp;
                        canFocusIn = true;
                    }

                    if (canFocusIn) {
                        if (stamp - lastStaticStamp > DELEY_DURATION) {
                            //移动后静止一段时间，可以发生对焦行为
                            if (!isFocusing) {
                                canFocusIn = false;
//                                onCameraFocus();
                                if (mCameraFocusListener != null) {
                                    mCameraFocusListener.onFocus();
                                }
//                                Log.i(TAG,"mobile focusing");
                            }
                        }
                    }

                    STATUE = STATUS_STATIC;
                }
            } else {
                lastStaticStamp = stamp;
                STATUE = STATUS_STATIC;
            }

            mX = x;
            mY = y;
            mZ = z;
        }
```


  [1]: http://blog.csdn.net/huweigoodboy/article/details/51378751
  [2]: http://on8vjlgub.bkt.clouddn.com/cameraPreview.png "cameraPreview"
  [3]: http://on8vjlgub.bkt.clouddn.com/%E6%B7%B1%E5%BA%A6%E6%88%AA%E5%9B%BE20171103003702.png "原理图"
