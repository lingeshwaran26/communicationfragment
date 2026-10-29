# Practical 6: Activity and Fragment Communication

Develop an Android application demonstrating bidirectional communication between an Activity and a Fragment using Bundles and an Interface callback.

---

## Project Overview

* **Project Name:** ActivityFragmentCommunication
* **Template:** Empty Views Activity
* **Language:** Java
* **Package Name:** `com.example.activityfragmentcommunication`
* **Fragment Creation:** `java package > New > Fragment > Fragment (Blank)`
  * **Fragment Name:** `DemoFragment`
  * **Layout Name:** `fragment_demo`

---

## 1. Java Source Files
**Path:** `app > java > com.example.activityfragmentcommunication >`

### I. `DemoFragment.java`
**Path:** `app > java > com.example.activityfragmentcommunication > DemoFragment.java`

```java
package com.example.activityfragmentcommunication;

import android.content.Context;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import android.widget.TextView;
import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

public class DemoFragment extends Fragment {

    TextView txtMessage;
    Button btnSend;
    String activityMessage;
    FragmentCommunication communication;

    public interface FragmentCommunication {
        void onMessageFromFragment(String message);
    }

    // Receive Activity context and attach listener interface
    @Override
    public void onAttach(@NonNull Context context) {
        super.onAttach(context);
        if (context instanceof FragmentCommunication) {
            communication = (FragmentCommunication) context;
        } else {
            throw new RuntimeException(context.toString() + " must implement FragmentCommunication");
        }
    }

    // Method to pass data from Activity via factory method
    public static DemoFragment newInstance(String message) {
        DemoFragment fragment = new DemoFragment();
        Bundle bundle = new Bundle();
        bundle.putString("msg", message);
        fragment.setArguments(bundle);
        return fragment;
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_demo, container, false);

        txtMessage = view.findViewById(R.id.txtMessage);
        btnSend = view.findViewById(R.id.btnSend);

        // Receiving data from Activity
        if (getArguments() != null) {
            activityMessage = getArguments().getString("msg");
            txtMessage.setText(activityMessage);
        }

        // Sending data to Activity via interface callback
        btnSend.setOnClickListener(v -> {
            if (communication != null) {
                communication.onMessageFromFragment("Hello Activity! Message from Fragment");
            }
        });

        return view;
    }
}
```

---

### II. `MainActivity.java`
**Path:** `app > java > com.example.activityfragmentcommunication > MainActivity.java`

```java
package com.example.activityfragmentcommunication;

import android.os.Bundle;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity implements DemoFragment.FragmentCommunication {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Sending data from Activity to Fragment
        DemoFragment fragment = DemoFragment.newInstance(
            "Hello Fragment! Message from Activity"
        );

        getSupportFragmentManager()
            .beginTransaction()
            .replace(R.id.fragment_container, fragment)
            .commit();
    }

    // Receiving data from Fragment
    @Override
    public void onMessageFromFragment(String message) {
        Toast.makeText(this, message, Toast.LENGTH_LONG).show();
    }
}
```

---

## 2. Layout XML Files
**Path:** `app > res > layout >`

### III. `activity_main.xml`
**Path:** `app > res > layout > activity_main.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout android:layout_height="match_parent" android:layout_width="match_parent" android:orientation="vertical" android:padding="16dp" xmlns:android="[http://schemas.android.com/apk/res/android](http://schemas.android.com/apk/res/android)">

    <TextView android:layout_gravity="center_horizontal" android:layout_height="wrap_content" android:layout_marginBottom="16dp" android:layout_width="wrap_content" android:text="Activity Screen" android:textSize="20sp" android:textStyle="bold"/>

    <FrameLayout android:id="@+id/fragment_container" android:layout_height="match_parent" android:layout_width="match_parent"/>

</LinearLayout>
```

---

### IV. `fragment_demo.xml`
**Path:** `app > res > layout > fragment_demo.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout android:background="#EDE7F6" android:gravity="center" android:layout_height="match_parent" android:layout_width="match_parent" android:orientation="vertical" android:padding="16dp" xmlns:android="[http://schemas.android.com/apk/res/android](http://schemas.android.com/apk/res/android)">

    <TextView android:id="@+id/txtMessage" android:layout_height="wrap_content" android:layout_marginBottom="24dp" android:layout_width="wrap_content" android:text="Waiting for message..." android:textColor="#311B92" android:textSize="18sp"/>

    <Button android:id="@+id/btnSend" android:layout_height="wrap_content" android:layout_width="wrap_content" android:text="Send to Activity"/>

</LinearLayout>
```

---

## 3. Communication Workflow

1. **Activity → Fragment:** 
   * `MainActivity` creates an instance of `DemoFragment` using the static factory method `newInstance(...)`.
   * The message is stored inside a `Bundle` argument.
   * `DemoFragment` extracts the string in `onCreateView()` via `getArguments().getString("msg")` and displays it in `txtMessage`.
2. **Fragment → Activity:**
   * `DemoFragment` defines a custom interface `FragmentCommunication`.
   * In `onAttach()`, it casts the host `MainActivity` context to `FragmentCommunication`.
   * When clicking `btnSend`, it triggers `communication.onMessageFromFragment(...)`, prompting `MainActivity` to show a `Toast`.

---

## 4. Execution Instructions

1. Start your Android Emulator or connect an active device.
2. Click the green **Play (Run)** button on the top toolbar in Android Studio.
3. Observe that the fragment displays: `"Hello Fragment! Message from Activity"`.
4. Click the **Send to Activity** button and verify that a Toast message appears showing: `"Hello Activity! Message from Fragment"`.
