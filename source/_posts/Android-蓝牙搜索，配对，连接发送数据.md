---
title: Android 蓝牙搜索，配对，连接发送数据
date: 2022-11-20 11:26:35
categories: Android
---

首先需要在清单配置里面添加两个权限:
```
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
```
android里面蓝牙是通过BluetoothAdapter来进行操作的，所以首先我们需要获取到BluetoothAdapter的实例
```
//先获取BlueToothAdapter的实例
BluetoothAdapter blueToothAdapter = BluetoothAdapter.getDefaultAdapter();
```
在搜索之前，我们可以先获取与我们配对过的设备
```
//获取已经配对过的设备的集合
Set<BluetoothDevice> bondedDevices = blueToothAdapter.getBondedDevices()
```
但是想要获取到配对过的设备，我们必须是在
```
//手机有蓝牙设备并且蓝牙是打开的
blueToothAdapter != null && blueToothAdapter.isEnabled()
```
下面我们谈一下蓝牙的<b>打开方式</b>，目前作者知道的方式有3种
第一种：
```
//强制打开蓝牙
blueToothAdapter.enable();
```
第二种：
```
//会以dialog的形式打开一个activity，并且如果我们通过startActivityForResult的形式的话
//还能查看蓝牙是否被打开，或者处理蓝牙被打开之后的操作
//如果是result_ok的话那么是打开，反之打开失败
startActivityForResult(new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE));
```
第三种：
```
//设置本地设备可以被其它设备搜索，可被搜索的时间是有限的，最多为300s
//效果和第二种类似
Intent discoverableIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_DISCOVERABLE);
discoverableIntent.putExtra(BluetoothAdapter.EXTRA_DISCOVERABLE_DURATION, 300);
startActivity(discoverableIntent);
```
<b>关闭蓝牙</b>我们可以调用
```
blueToothAdapter.disable();
```
接下来是<b>蓝牙搜索</b>
首先如果想要开启蓝牙搜索，那么只需要调用
```
blueToothAdapter.startDiscovery();
```
但是我们该如何接收我们搜索到的设备呢，很明显当然是通过接收广播的形式来接收。所以我们应该自定义一个广播接收器，
```
class BluetoothReceiver extends BroadcastReceiver {

        @Override
        public void onReceive(Context context, Intent intent) {

            if (intent.getAction().equals(BluetoothDevice.ACTION_FOUND)) {
                Log.e(getPackageName(), "找到新设备了");
                BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
            }
        }
    }
```
如上代码，我们可以获取到搜索到的蓝牙设备。我们只需在代码里面注册这个广播接收器，在调用蓝牙的搜索方法，就能够进行蓝牙的搜索了。
```
//注册广播
IntentFilter intentFilter = new IntentFilter();
intentFilter.addAction(BluetoothDevice.ACTION_FOUND);
registerReceiver(new BluetoothReceiver(), intentFilter);
```
这里需要注意的是，如果你的代码将运行在（Build.VERSION.SDK_INT >= 23）的设备上，那么务必加上以下权限，并在代码中动态的申请权限
```
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
```
```
private void requestPermission() {
        if (Build.VERSION.SDK_INT >= 23) {
            int checkAccessFinePermission = ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION);
            if (checkAccessFinePermission != PackageManager.PERMISSION_GRANTED) {
                ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.ACCESS_FINE_LOCATION},
                        REQUEST_PERMISSION_ACCESS_LOCATION);
                Log.e(getPackageName(), "没有权限，请求权限");
                return;
            }
            Log.e(getPackageName(), "已有定位权限");
            //这里可以开始搜索操作
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
        switch (requestCode) {
            case REQUEST_PERMISSION_ACCESS_LOCATION: {
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    Log.e(getPackageName(), "开启权限permission granted!");
                    //这里可以开始搜索操作
                } else {
                    Log.e(getPackageName(), "没有定位权限，请先开启!");
                }
            }
        }
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    }
```
再来就是<b>蓝牙的配对</b>，目前楼主采用的是通过反射的形式来调用BluetoothDevice的createBondde()方法，如果我们想监听配对的这个过程，那么我们可以为广播接收器再注册一个action。
```
try {
      //如果想要取消已经配对的设备，只需要将creatBond改为removeBond
       Method method = BluetoothDevice.class.getMethod("createBond");
       Log.e(getPackageName(), "开始配对");
       method.invoke(device);
    } catch (Exception e) {
      e.printStackTrace();
    }
```
```
//绑定状态发生变化
intentFilter.addAction(BluetoothDevice.ACTION_BOND_STATE_CHANGED);
```
广播接收器里面我们可以这样写
```
if (intent.getAction().equals(BluetoothDevice.ACTION_BOND_STATE_CHANGED)) {
                BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
                switch (device.getBondState()) {
                    case BluetoothDevice.BOND_NONE:
                        Log.e(getPackageName(), "取消配对");
                        break;
                    case BluetoothDevice.BOND_BONDING:
                        Log.e(getPackageName(), "配对中");
                        break;
                    case BluetoothDevice.BOND_BONDED:
                        Log.e(getPackageName(), "配对成功");
                        break;
                }
}
```
再来就是<b>向已经配对的设备发送数据</b>
发送数据分为服务端和客户端，通过socket来进行消息的交互。

服务端
```
new Thread(new Runnable() {
            @Override
            public void run() {
                InputStream is = null;
                try {
                    BluetoothServerSocket serverSocket = blueToothAdapter.listenUsingRfcommWithServiceRecord("serverSocket", uuid);
                    mHandler.sendEmptyMessage(startService);
                    BluetoothSocket accept = serverSocket.accept();
                    is = accept.getInputStream();

                    byte[] bytes = new byte[1024];
                    int length = is.read(bytes);

                    Message msg = new Message();
                    msg.what = getMessageOk;
                    msg.obj = new String(bytes, 0, length);
                    mHandler.sendMessage(msg);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }).start();
```

客户端
```
new Thread(new Runnable() {
            @Override
            public void run() {
                OutputStream os = null;
                try {
                    BluetoothSocket socket = strArr.get(i).createRfcommSocketToServiceRecord(uuid);
                    socket.connect();
                    os = socket.getOutputStream();
                    os.write("testMessage".getBytes());
                    os.flush();
                    mHandler.sendEmptyMessage(sendOver);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
```
可以看到无论是服务端还是客户端，都需要新起一个子线程来操作。那么服务端和客户端是怎么识别对方的呢，那么就需要用到<b>UUID</b>了，只有当服务端和客户端的<b>UUID</b>相同的时候才能够建立连接。

<b>蓝牙连接发送数据的时候的坑</b>
当博主第一次写好客户端服务端测试的时候，服务端一直报错
```
java.io.IOException: bt socket closed, read return: -1
```
也就是这句话报错
```
is.read(bytes)
```
当时一直以为是读取返回值是-1就会报错，但是不对啊，以前这么写也没错过，在网上百度了半天，看了别人的博客论坛也没有解决办法，最后才注意到报错的前一句，
```
bt socket closed
```
之前一直认为是什么服务端这边，报错连接才会断开，后来一想客户端也会断开连接啊，这才找到问题所在。原来是我当时write完数据之后将连接关闭了，这才导致服务端这边连接断开的。所以重要的事情要说3遍，千万别急着断开连接，千万别急着断开连接，千万别急着断开连接。

<a href="https://github.com/MonkZl/BlueToothDemo">github源代码链接</a>
