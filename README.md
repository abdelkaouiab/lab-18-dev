# 📱 Lab 18 — BroadcastReceiver en Android (Java)

## 🧭 Aperçu

Ce lab permet de comprendre et maîtriser les **BroadcastReceiver** en Android, composants essentiels pour réagir aux événements système ou internes à l’application.

L’application **ReceiverDemo** implémente :

* Un **Receiver dynamique** pour le mode avion
* Un **Receiver statique** pour le démarrage du téléphone
* Un **Broadcast personnalisé (custom)** envoyé et reçu dans l’application

---

## 🎯 Objectifs

* Comprendre le fonctionnement des BroadcastReceiver
* Différencier **statique vs dynamique**
* Manipuler `Intent`, `IntentFilter`
* Gérer les permissions et restrictions Android modernes
* Appliquer les bonnes pratiques (performance, sécurité)

---

## 🏗️ Étape 1 : Création du projet

* Nom : `ReceiverDemo`
* Langage : Java
* Min SDK : API 24

---

## ✈️ Étape 2 : Receiver Dynamique (Mode Avion)

### 📌 Classe : `AirplaneModeReceiver.java`

```java
package com.example.receiverdemo;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

public class AirplaneModeReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        if (Intent.ACTION_AIRPLANE_MODE_CHANGED.equals(intent.getAction())) {
            boolean isAirplaneOn = intent.getBooleanExtra("state", false);

            String message = isAirplaneOn
                    ? "Mode Avion ACTIVÉ - Plus de connexion !"
                    : "Mode Avion DÉSACTIVÉ - Connexions rétablies";

            Toast.makeText(context, message, Toast.LENGTH_LONG).show();
        }
    }
}
```

### 📌 Points importants

* `onReceive()` → exécuté sur le **thread principal**
* Ne pas faire d’opérations lourdes
* Utilise `Intent.ACTION_AIRPLANE_MODE_CHANGED`

---

## 🔌 Étape 3 : Receiver Statique (BOOT)

### 📌 Classe : `BootReceiver.java`

```java
package com.example.receiverdemo;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

public class BootReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if (Intent.ACTION_BOOT_COMPLETED.equals(intent.getAction())) {
            Toast.makeText(context, "Téléphone démarré - Receiver statique activé !", Toast.LENGTH_LONG).show();
        }
    }
}
```

---

## 📜 Étape 4 : Manifest & Permissions

### 📌 Permission

```xml
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```

### 📌 Déclaration des Receivers

```xml
<application>

    <receiver
        android:name=".BootReceiver"
        android:exported="false">
        <intent-filter>
            <action android:name="android.intent.action.BOOT_COMPLETED" />
        </intent-filter>
    </receiver>

    <receiver
        android:name=".CustomEventReceiver"
        android:exported="false" />

</application>
```

### ⚠️ Important

* `exported="false"` obligatoire (sécurité Android 12+)
* Receiver dynamique **non déclaré** dans le Manifest

---

## 🎮 Étape 5 : MainActivity

```java
package com.example.receiverdemo;

import android.content.Intent;
import android.content.IntentFilter;
import android.os.Bundle;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    private AirplaneModeReceiver airplaneReceiver;
    private boolean isReceiverRegistered = false;
    private Button btnToggleAirplane, btnSendCustom;
    private TextView tvStatus;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        airplaneReceiver = new AirplaneModeReceiver();
        tvStatus = findViewById(R.id.tvStatus);
        btnToggleAirplane = findViewById(R.id.btnToggleAirplane);
        btnSendCustom = findViewById(R.id.btnSendCustom);

        btnToggleAirplane.setOnClickListener(v -> toggleAirplaneReceiver());
        btnSendCustom.setOnClickListener(v -> sendCustomBroadcast());
    }

    private void toggleAirplaneReceiver() {
        if (!isReceiverRegistered) {
            IntentFilter filter = new IntentFilter();
            filter.addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED);

            registerReceiver(airplaneReceiver, filter);
            isReceiverRegistered = true;

            tvStatus.setText("Receiver Mode Avion : ACTIVÉ (dynamique)");
            btnToggleAirplane.setText("Désactiver Receiver Avion");
        } else {
            unregisterReceiver(airplaneReceiver);
            isReceiverRegistered = false;

            tvStatus.setText("Receiver Mode Avion : DÉSACTIVÉ");
            btnToggleAirplane.setText("Activer Receiver Avion");
        }
    }

    private void sendCustomBroadcast() {
        Intent intent = new Intent("com.example.receiverdemo.CUSTOM_EVENT");
        intent.putExtra("message", "Bonjour depuis le custom broadcast !");

        sendBroadcast(intent);

        Toast.makeText(this, "Custom Broadcast envoyé !", Toast.LENGTH_SHORT).show();
    }

    @Override
    protected void onDestroy() {
        if (isReceiverRegistered) {
            unregisterReceiver(airplaneReceiver);
        }
        super.onDestroy();
    }
}
```

---

## 📡 Étape 6 : Custom Broadcast Receiver

```java
package com.example.receiverdemo;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

public class CustomEventReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if ("com.example.receiverdemo.CUSTOM_EVENT".equals(intent.getAction())) {
            String message = intent.getStringExtra("message");
            Toast.makeText(context, "Custom reçu : " + message, Toast.LENGTH_LONG).show();
        }
    }
}
```

---

## 🎨 Étape 7 : Layout

```xml
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:id="@+id/tvStatus"
        android:text="Status"
        android:textSize="18sp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <Button
        android:id="@+id/btnToggleAirplane"
        android:text="Activer Receiver Avion"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <Button
        android:id="@+id/btnSendCustom"
        android:text="Envoyer Custom Broadcast"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

</LinearLayout>
```

---

## 🧪 Étape 8 : Tests

### ✔️ Mode Avion

* Activer le receiver
* Activer/Désactiver mode avion
* Observer le Toast

### ✔️ Custom Broadcast

* Cliquer sur bouton
* Observer "Custom reçu"

### ✔️ Boot

```bash
adb reboot
```

---

## 🧠 Conclusion

### 🔁 Types de Receiver

* **Dynamique** → recommandé (économie batterie)
* **Statique** → événements système rares

### 🔐 Bonnes pratiques

* Toujours `exported=false`
* Nettoyer avec `unregisterReceiver()`
* Éviter les tâches lourdes dans `onReceive()`

### 🚀 Bonus

* Utiliser `LocalBroadcastManager` pour communication interne sécurisée

---

## ✅ Résultat

Vous maîtrisez maintenant :

* Les BroadcastReceiver
* Les événements système Android
* La communication interne via broadcast
