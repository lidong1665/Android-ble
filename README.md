
Android-ble

Android蓝牙4.0操作demo 最近，随着智能穿戴式设备、智能医疗以及智能家居的普及，蓝牙开发在移动开中显得非常的重要。由于公司需要，研究了一下，蓝牙4.0在Android中的应用。以下是我的一些总结。

## 1.先介绍一下关于蓝牙4.0中的一些名词吧：
* (1)GATT(Gneric Attibute Profile)
通过ble连接，读写属性类小数据Profile通用的规范。现在所有的ble应用Profile 都是基于GATT
* (2)ATT(Attribute Protocal) GATT是基于ATT Potocal的ATT针对BLE设备专门做的具体就是传输过程中使用尽量少的数据，每个属性都有个唯一的UUID，属性chartcteristics and Service的形式传输。

* (3)Service是Characteristic的集合。
*  (4).Characteristic 特征类型。

比如。有个蓝牙ble的血压计。他可能包括多个Service，每个Service有包括多个Characteristic

注意：蓝牙ble只能支持Android 4.3以上的系统 SDK>=18

## 2.以下是开发的步骤：
### 2.1首先获取BluetoothManager
> BluetoothManager bluetoothManager = (BluetoothManager) getSystemService(Context.BLUETOOTH_SERVICE);
### 2.2获取BluetoothAdapter

```
BluetoothAdapter mBluetoothAdapter = bluetoothManager.getAdapter();
```
### 2.3创建BluetoothAdapter.LeScanCallback

```
private BluetoothAdapter.LeScanCallback mLeScanCallback = new BluetoothAdapter.LeScanCallback() {

    @Override  
    public void onLeScan(final BluetoothDevice device, int rssi, final byte[] scanRecord) {  

        runOnUiThread(new Runnable() {  
            @Override  
            public void run() {  
                try {  
                    String struuid = NumberUtils.bytes2HexString(NumberUtils.reverseBytes(scanRecord)).replace("-", "").toLowerCase();  
                    if (device!=null && struuid.contains(DEVICE_UUID_PREFIX.toLowerCase())) {  
                        mBluetoothDevices.add(device);  
                    }  
                } catch (Exception e) {  
                    e.printStackTrace();  
                }  
            }  
        });  
    }  
};  
```

### 2.4.开始搜索设备。

```
mBluetoothAdapter.startLeScan(mLeScanCallback);
```

### 2.5.BluetoothDevice 描述了一个蓝牙设备 提供了getAddress()设备Mac地址,getName()设备的名称。

### 2.6开始连接设备

```
public boolean connect(final String address) {
if (mBluetoothAdapter == null || address == null) {
Log.w(TAG, "BluetoothAdapter not initialized or unspecified address.");
return false;
}

    // Previously connected device. Try to reconnect. (先前连接的设备。 尝试重新连接)  
    if (mBluetoothDeviceAddress != null && address.equals(mBluetoothDeviceAddress) && mBluetoothGatt != null) {  
        Log.d(TAG, "Trying to use an existing mBluetoothGatt for connection.");  
        if (mBluetoothGatt.connect()) {  
            mConnectionState = STATE_CONNECTING;  
            return true;  
        } else {  
            return false;  
        }  
    }  

    final BluetoothDevice device = mBluetoothAdapter.getRemoteDevice(address);  
    if (device == null) {  
        Log.w(TAG, "Device not found.  Unable to connect.");  
        return false;  
    }  
    // We want to directly connect to the device, so we are setting the  
    // autoConnect  
    // parameter to false.  
    mBluetoothGatt = device.connectGatt(this, false, mGattCallback);  
    Log.d(TAG, "Trying to create a new connection.");  
    mBluetoothDeviceAddress = address;  
    mConnectionState = STATE_CONNECTING;  
    return true;  
}  
```

### 2.7连接到设备之后获取设备的服务(Service)和服务对应的Characteristic。

```
// Demonstrates how to iterate through the supported GATT
// Services/Characteristics.
// In this sample, we populate the data structure that is bound to the
// ExpandableListView
// on the UI.
private void displayGattServices(List gattServices) {
if (gattServices == null)
return;
String uuid = null;
ArrayList> gattServiceData = new ArrayList<>();
ArrayList>> gattCharacteristicData = new ArrayList<>();

mGattCharacteristics = new ArrayList<>();  

// Loops through available GATT Services.  
for (BluetoothGattService gattService : gattServices) {  
    HashMap<String, String> currentServiceData = new HashMap<>();  
    uuid = gattService.getUuid().toString();  
    if (uuid.contains("ba11f08c-5f14-0b0d-1080")) {//服务的uuid  
        //System.out.println("this gattService UUID is:" + gattService.getUuid().toString());  
        currentServiceData.put(LIST_NAME, "Service_OX100");  
        currentServiceData.put(LIST_UUID, uuid);  
        gattServiceData.add(currentServiceData);  
        ArrayList<HashMap<String, String>> gattCharacteristicGroupData = new ArrayList<>();  
        List<BluetoothGattCharacteristic> gattCharacteristics = gattService.getCharacteristics();  
        ArrayList<BluetoothGattCharacteristic> charas = new ArrayList<>();  

        // Loops through available Characteristics.  
        for (BluetoothGattCharacteristic gattCharacteristic : gattCharacteristics) {  
            charas.add(gattCharacteristic);  
            HashMap<String, String> currentCharaData = new HashMap<>();  
            uuid = gattCharacteristic.getUuid().toString();  
            if (uuid.toLowerCase().contains("cd01")) {  
                currentCharaData.put(LIST_NAME, "cd01");  
            } else if (uuid.toLowerCase().contains("cd02")) {  
                currentCharaData.put(LIST_NAME, "cd02");  
            } else if (uuid.toLowerCase().contains("cd03")) {  
                currentCharaData.put(LIST_NAME, "cd03");  
            } else if (uuid.toLowerCase().contains("cd04")) {  
                currentCharaData.put(LIST_NAME, "cd04");  
            } else {  
                currentCharaData.put(LIST_NAME, "write");  
            }  

            currentCharaData.put(LIST_UUID, uuid);  
            gattCharacteristicGroupData.add(currentCharaData);  
        }  

        mGattCharacteristics.add(charas);  

        gattCharacteristicData.add(gattCharacteristicGroupData);  

        mCharacteristicCD01 = gattService.getCharacteristic(UUID.fromString("0000cd01-0000-1000-8000-00805f9b34fb"));  
        mCharacteristicCD02 = gattService.getCharacteristic(UUID.fromString("0000cd02-0000-1000-8000-00805f9b34fb"));  
        mCharacteristicCD03 = gattService.getCharacteristic(UUID.fromString("0000cd03-0000-1000-8000-00805f9b34fb"));  
        mCharacteristicCD04 = gattService.getCharacteristic(UUID.fromString("0000cd04-0000-1000-8000-00805f9b34fb"));  
        mCharacteristicWrite = gattService.getCharacteristic(UUID.fromString("0000cd20-0000-1000-8000-00805f9b34fb"));  

        //System.out.println("=======================Set Notification==========================");  
        // 开始顺序监听，第一个：CD01  
        mBluetoothLeService.setCharacteristicNotification(mCharacteristicCD01, true);  
        mBluetoothLeService.setCharacteristicNotification(mCharacteristicCD02, true);  
        mBluetoothLeService.setCharacteristicNotification(mCharacteristicCD03, true);  
        mBluetoothLeService.setCharacteristicNotification(mCharacteristicCD04, true);  
    }  
}  
}
```

### 2.8获取到特征之后，找到服务中可以向下位机写指令的特征，向该特征写入指令。

public void wirteCharacteristic(BluetoothGattCharacteristic characteristic) {

    if (mBluetoothAdapter == null || mBluetoothGatt == null) {  
        Log.w(TAG, "BluetoothAdapter not initialized");  
        return;  
    }  

    mBluetoothGatt.writeCharacteristic(characteristic);  

}  
### 2.9写入成功之后，开始读取设备返回来的数据。

```
private final BluetoothGattCallback mGattCallback = new BluetoothGattCallback() {
@Override
public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) {
String intentAction;
//System.out.println("=======status:" + status);
if (newState == BluetoothProfile.STATE_CONNECTED) {
intentAction = ACTION_GATT_CONNECTED;
mConnectionState = STATE_CONNECTED;
broadcastUpdate(intentAction);
Log.i(TAG, "Connected to GATT server.");
// Attempts to discover services after successful connection.
Log.i(TAG, "Attempting to start service discovery:" + mBluetoothGatt.discoverServices());

        } else if (newState == BluetoothProfile.STATE_DISCONNECTED) {  
            intentAction = ACTION_GATT_DISCONNECTED;  
            mConnectionState = STATE_DISCONNECTED;  
            Log.i(TAG, "Disconnected from GATT server.");  
            broadcastUpdate(intentAction);  
        }  
    }  

    @Override  
    public void onServicesDiscovered(BluetoothGatt gatt, int status) {  
        if (status == BluetoothGatt.GATT_SUCCESS) {  
            broadcastUpdate(ACTION_GATT_SERVICES_DISCOVERED);  
        } else {  
            Log.w(TAG, "onServicesDiscovered received: " + status);  
        }  
    }  
    //从特征中读取数据  
    @Override  
    public void onCharacteristicRead(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic, int status) {  
        //System.out.println("onCharacteristicRead");  
        if (status == BluetoothGatt.GATT_SUCCESS) {  
            broadcastUpdate(ACTION_DATA_AVAILABLE, characteristic);  
        }  
    }  
    //向特征中写入数据  
    @Override  
    public void onCharacteristicWrite(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic, int status) {  
        //System.out.println("--------write success----- status:" + status);  
    }  

    /* 
     * when connected successfully will callback this method this method can 
     * dealwith send password or data analyze 

     *当连接成功将回调该方法 
     */  
    @Override  
    public void onCharacteristicChanged(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic) {  
        broadcastUpdate(ACTION_DATA_AVAILABLE, characteristic);  
        if (characteristic.getValue() != null) {  

            //System.out.println(characteristic.getStringValue(0));  
        }  
        //System.out.println("--------onCharacteristicChanged-----");  
    }  

    @Override  
    public void onDescriptorWrite(BluetoothGatt gatt, BluetoothGattDescriptor descriptor, int status) {  

        //System.out.println("onDescriptorWriteonDescriptorWrite = " + status + ", descriptor =" + descriptor.getUuid().toString());  

        UUID uuid = descriptor.getCharacteristic().getUuid();  
        if (uuid.equals(UUID.fromString("0000cd01-0000-1000-8000-00805f9b34fb"))) {  
            broadcastUpdate(ACTION_CD01NOTIDIED);  
        } else if (uuid.equals(UUID.fromString("0000cd02-0000-1000-8000-00805f9b34fb"))) {  
            broadcastUpdate(ACTION_CD02NOTIDIED);  
        } else if (uuid.equals(UUID.fromString("0000cd03-0000-1000-8000-00805f9b34fb"))) {  
            broadcastUpdate(ACTION_CD03NOTIDIED);  
        } else if (uuid.equals(UUID.fromString("0000cd04-0000-1000-8000-00805f9b34fb"))) {  
            broadcastUpdate(ACTION_CD04NOTIDIED);  
        }  
    }  

    @Override  
    public void onReadRemoteRssi(BluetoothGatt gatt, int rssi, int status) {  
        //System.out.println("rssi = " + rssi);  
    }  
};  

----------------------------------------------  
  //从特征中读取数据  
    @Override  
    public void onCharacteristicRead(BluetoothGatt gatt, BluetoothGattCharacteristic characteristic, int status) {  
        //System.out.println("onCharacteristicRead");  
        if (status == BluetoothGatt.GATT_SUCCESS) {  
            broadcastUpdate(ACTION_DATA_AVAILABLE, characteristic);  
        }  
    }  
2.10、断开连接

/** * Disconnects an existing connection or cancel a pending connection. The * disconnection result is reported asynchronously through the * {@code BluetoothGattCallback#onConnectionStateChange(android.bluetooth.BluetoothGatt, int, int)} * callback. */
public void disconnect() {
if (mBluetoothAdapter == null || mBluetoothGatt == null) {
Log.w(TAG, "BluetoothAdapter not initialized");
return;
}
mBluetoothGatt.disconnect();
}
```

### 2.11、数据的转换方法

```
// byte转十六进制字符串
public static String bytes2HexString(byte[] bytes) {
String ret = "";
for (byte aByte : bytes) {
String hex = Integer.toHexString(aByte & 0xFF);
if (hex.length() == 1) {
hex = '0' + hex;
}
ret += hex.toUpperCase(Locale.CHINA);
}
return ret;
}

/** * 将16进制的字符串转换为字节数组 *
* @param message * @return 字节数组 */
public static byte[] getHexBytes(String message) {
int len = message.length() / 2;
char[] chars = message.toCharArray();
String[] hexStr = new String[len];
byte[] bytes = new byte[len];
for (int i = 0, j = 0; j < len; i += 2, j++) {
hexStr[j] = "" + chars[i] + chars[i + 1];
bytes[j] = (byte) Integer.parseInt(hexStr[j], 16);
}
return bytes;
}
```


[代码下载地址：https://github.com/lidong1665/Android-ble](https://github.com/lidong1665/Android-ble)


### 总结

大概整体就是如上的步骤。但是也是要具体根据厂家的协议来实现通信的过程。

就那一个我们项目中的demo说一下。 一个蓝牙ble的血压计。 上位机---手机 下位机 -- 血压计 1.血压计与手机连接蓝牙之后。 2.上位机主动向下位机发送一个身份验证指令，下位机收到指令后开始给上位做应答， 3.应答成功，下位机会将测量的血压数据传送到上位机。 4.最后断开连接。


