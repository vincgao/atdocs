# Input


Input user drivers provide an interface for apps to inject events into Android's [input pipeline](http://source.android.google.cn/devices/input/overview.html). With this API, apps can emulate a Human Interface Device (HID) or connect external hardware to the input system using [Peripheral I/O](https://developer.android.google.cn/things/sdk/pio/index.html).

## Key Events

* * *

Key events indicate a momentary press and release of an input switch. They are generally used for generic button input (e.g. volume keys, media playback keys) and the keys of a keyboard. Android represents each event as a [KeyEvent](https://developer.android.google.cn/reference/android/view/KeyEvent.html) instance.

1.  Create a new driver instance using the `InputDriver.Builder` and the source type `SOURCE_CLASS_BUTTON`.
2.  Register the driver with the `UserDriverManager`.

        import com.google.android.things.userdriver.InputDriver;import com.google.android.things.userdriver.UserDriverManager;...public class ButtonDriverService extends Service {    // Driver parameters    private static final String DRIVER_NAME = "EscapeButton";    private static final int DRIVER_VERSION = 1;    // Key code for driver to emulate    private static final int KEY_CODE = KeyEvent.KEYCODE_ESCAPE;    private InputDriver mDriver;    @Override    public void onCreate() {        super.onCreate();        // Create a new driver instance        mDriver = InputDriver.builder(InputDevice.SOURCE_CLASS_BUTTON)                .setName(DRIVER_NAME)                .setVersion(DRIVER_VERSION)                .setKeys(new int[] {KEY_CODE})                .build();        // Register with the framework        UserDriverManager manager = UserDriverManager.getManager();        manager.registerInputDriver(mDriver);    }    @Override    public IBinder onBind(Intent intent) {        return null;    }}

3.  When a hardware event occurs, construct a new `KeyEvent` for each state chnage with the current key code and input action.

4.  Inject the events into the driver with the `emit()` method.

        public class ButtonDriverService extends Service {    ...    // A state change has occurred    private void triggerEvent(boolean pressed) {        int action = pressed ? KeyEvent.ACTION_DOWN : KeyEvent.ACTION_UP;        KeyEvent[] events = new KeyEvent[] {new KeyEvent(action, KEY_CODE)};        if (!mDriver.emit(events)) {            Log.w(TAG, "Unable to emit key event");        }    }}

5.  Unregister the driver when key events are not longer required.

        public class ButtonDriverService extends Service {    ...    @Override    public void onDestroy() {        super.onDestroy();        UserDriverManager manager = UserDriverManager.getManager();        manager.unregisterInputDriver(mDriver);    }}

## Motion Events

* * *

Input drivers can also emit motion events to connect a pointing device to the framework, such as a touchpad or mouse. These devices report an absolute position value as an x/y coordinate. Each event include an optional pressed state to indicate if the event represents a "tap" or "click" event at that location.

<aside class="note">**Note:** <span>You must report all coordinates as positive integer values.</span></aside>

1.  Create a new driver instance using the `InputDriver.Builder` and the source type `SOURCE_TOUCHPAD`.
2.  Register the driver with the `UserDriverManager`.

        import com.google.android.things.userdriver.InputDriver;...public class TouchpadDriverService extends Service {    // Driver parameters    private static final String DRIVER_NAME = "Touchpad";    private static final int DRIVER_VERSION = 1;    private InputDriver mDriver;    @Override    public void onCreate() {        super.onCreate();        mDriver = InputDriver.builder(InputDevice.SOURCE_TOUCHPAD)                .setName(DRIVER_NAME)                .setVersion(DRIVER_VERSION)                .setAbsMax(MotionEvent.AXIS_X, 255)                .setAbsMax(MotionEvent.AXIS_Y, 255)                .build();        UserDriverManager manager = UserDriverManager.getManager();        manager.registerInputDriver(mDriver);    }    @Override    public IBinder onBind(Intent intent) {        return null;    }}

3.  When a hardware event occurs, inject the new coordinates into the driver with the `emit()` method.

        public class TouchpadDriverService extends Service {    ...    // A state change has occurred    private void triggerEvent(int x, int y, boolean pressed) {        if (!mDriver.emit(x, y, pressed)) {            Log.w(TAG, "Unable to emit motion event");        }    }}

4.  Unregister the driver when pointer events are not longer required.

        public class TouchpadDriverService extends Service {    ...    @Override    public void onDestroy() {        super.onDestroy();        UserDriverManager manager = UserDriverManager.getManager();        manager.unregisterInputDriver(mDriver);    }}

## Handling Input Events

* * *

Android delivers input events to the foreground activity through various callback methods. Your app receives key events through the `onKeyDown()` and `onKeyUp()` methods, and all other input events through the `onGenericMotionEvent()` method.

    public class HomeActivity extends Activity {    @Override    public boolean onKeyDown(int keyCode, KeyEvent event) {        // Handle key pressed and repeated events        return true;    }    @Override    public boolean onKeyUp(int keyCode, KeyEvent event) {        // Handle key released events        return true;    }    @Override    public boolean onGenericMotionEvent(MotionEvent event) {        // Handle motion input events        return true;    }}

See [Handling Controller Actions](https://developer.android.google.cn/training/game-controllers/controller-input.html) for more details on how Android handles input events from external source devices.

## Adding the required permission

* * *

Add the required permission for the user driver to your app's manifest file:

        <uses-permission android:name="com.google.android.things.permission.MANAGE_INPUT_DRIVERS" />

