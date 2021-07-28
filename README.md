---
title: Identity Vault Getting Started with Angular
sidebar_label: Angular
---

# Getting Started with Identity Vault in @ionic/angular

This application walks through the basic setup and use of Ionic's Identity Vault in an `@ionic/angular` application.

The most common use case of Identity Vault is to connect to a back end service and store user session data. For the purpose of this tutorial, the application we build will not connect to an actual service. Instead, the application will store information that the user enters.

- `src/app/vault.service.ts`: An Angular service that abstracts the logic associated with using Identity Vault. Methods and properties here model what might be done in a real application.

- `src/app/home/home.page.ts`: The main view will have several form controls that allow the user to manipulate the vault. An application would not typically do this. Rather, it would call the methods from `vault-service.ts` within various workflows. In this "getting started" demo application, however, this allows us to easily play around with the various APIs to see how they behave.

## Generate the Application

The first step to take is to generate our application:

```bash
ionic start getting-started-iv-angular blank --type=angular --capacitor
```

Now that the application has been generated let's add the iOS and Android platforms.

Open the `capacitor.config.ts` file and change the `appId` to something unique like `io.ionic.gettingstartedivangular`:

```TypeScript
import { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'io.ionic.gettingstartedivangular',
  appName: 'getting-started-iv-angular',
  webDir: 'www',
  bundledWebRuntime: false
};

export default config;
```

Next, build the application and install and create the platforms:

```bash
npm run build
ionic cap add android
ionic cap add ios
```

Finally, in order to ensure that the web application bundle is copied over each build, add `cap copy` to the `build` script in the `package.json` file:

```JSON
"scripts": {
  "build": "ng build && cap copy",
  ...
},
```

## Install Identity Vault

In order to install Identity Vault you will need to use `ionic enterprise register` to register your product key. This will create a `.npmrc` file containing the product key.

If you have already performed that step for your production application, you can just copy the `.npmrc` file from your production project. Since this application is for learning purposes only, you don't need to obtain another key.

You can now install Identity Vault:

```bash
npm install @ionic-enterprise/identity-vault
```

## Create the Vault

In this step, we will create the vault and test it by storing and retrieving a value from it. This value will be called `session`, since storing session data in a vault is the most common use case of Identity Vault. However, it is certainly not the _only_ use case.

First, create a service named vault:

```bash
ionic generate service vault
```

This will create a file named `src/app/vault.service.ts`. Within this file, we will define the vault as well as create an Angular service that abstracts all of the logic we need in order to interact with the vault:

```TypeScript
import { Injectable } from '@angular/core';
import { Capacitor } from '@capacitor/core';
import {
  BrowserVault,
  DeviceSecurityType,
  IdentityVaultConfig,
  Vault,
  VaultType,
} from '@ionic-enterprise/identity-vault';

export interface VaultServiceState {
  session: string;
}

@Injectable({
  providedIn: 'root'
})
export class VaultService {

  config: IdentityVaultConfig = {
    key: 'io.ionic.getstartedivangular',
    type: VaultType.SecureStorage,
    deviceSecurityType: DeviceSecurityType.None,
    lockAfterBackgrounded: 2000,
    shouldClearVaultAfterTooManyFailedAttempts: true,
    customPasscodeInvalidUnlockAttempts: 2,
    unlockVaultOnLoad: false,
  };

  key = 'sessionData';

  vault: Vault;

  public state: VaultServiceState = {
    session: ''
  };

  constructor() {
    this.init();
  }

  async init() {
    this.vault = Capacitor.getPlatform() === 'web'
      ? new BrowserVault(this.config)
      : new Vault(config);
  }

  async setSession(value: string): Promise<void> {
    this.state.session = value;
    await this.vault.setValue(this.key, value);
  }

  async restoreSession() {
    const value = await this.vault.getValue(this.key);
    this.state.session = value;
  }
}
```

Let's look at this file section by section:

The first thing we do is define a configuration for our vault. The `key` gives the vault a name. The other properties provide a default behavior for our vault. As we shall see later, the configuration can be changed as we use the vault.

```TypeScript
config: IdentityVaultConfig = {
  key: 'io.ionic.getstartedivangular',
  type: VaultType.SecureStorage,
  deviceSecurityType: DeviceSecurityType.None,
  lockAfterBackgrounded: 2000,
  shouldClearVaultAfterTooManyFailedAttempts: true,
  customPasscodeInvalidUnlockAttempts: 2,
  unlockVaultOnLoad: false,
};

key = 'sessionData';
```

Next, we define a state object to store our `session` data.

```TypeScript
public state: VaultServiceState = {
  session: ''
};
```

Then, we initialize the vault.

```TypeScript
constructor() {
  this.init();
}

async init() {
  this.vault = Capacitor.getPlatform() === 'web'
    ? new BrowserVault(this.config)
    : new Vault(config);
}
```

:::note
The `BrowserVault` class allows developers to use their normal web-based development workflow. It does **not** provide locking or security functionality.
:::

The service additionally contains methods store and restore our session.

```TypeScript
async setSession(value: string): Promise<void> {
  this.state.session = value;
  await this.vault.setValue(this.key, value);
}

async restoreSession() {
  const value = await this.vault.getValue(this.key);
  this.state.session = value;
}
```

:::note
It's recommended to abstract vault functionality into functions that define how the rest of the application should interact with the vault instead of directly returning the `vault` instance. This creates more maintainable and debuggable code, while preventing the rest of the code from being exposed to potential API changes and reduces the chance of duplicating code.
:::

Now that we have the vault in place, let's switch over to `src/app/home/home.page.html` and implement some simple interactions with the vault. Here is a snapshot of what we will change:

1. Replace the "container" `div` with a list of form controls.
2. Add a `setup()` function.
3. Remove the existing styling.

Update the `HomePage` component to match the following code:

```HTML
<ion-header [translucent]="true">
  <ion-toolbar>
    <ion-title>
      Blank
    </ion-title>
  </ion-toolbar>
</ion-header>

<ion-content [fullscreen]="true">
  <ion-header collapse="condense">
    <ion-toolbar>
      <ion-title size="large">Blank</ion-title>
    </ion-toolbar>
  </ion-header>

  <ion-list>
    <ion-item>
      <ion-label position="floating">Enter the "session" data</ion-label>
      <ion-input [(ngModel)]="state.session"></ion-input>
    </ion-item>

    <ion-item>
      <ion-label>
        <ion-button expand="block" (click)="setSession(state.session)">Set Session Data</ion-button>
      </ion-label>
    </ion-item>

    <ion-item>
      <ion-label>
        <ion-button expand="block" (click)="restoreSession()">Restore Session Data</ion-button>
      </ion-label>
    </ion-item>

    <ion-item>
      <ion-label>
        <div>Session Data: {{ state.session }}</div>
      </ion-label>
    </ion-item>
  </ion-list>
</ion-content>
```

Update `src/app/home/home.page.ts` to match the following code:

```TypeScript
import { Component } from '@angular/core';
import { VaultService, VaultServiceState } from '../vault.service';

@Component({
  selector: 'app-home',
  templateUrl: 'home.page.html',
  styleUrls: ['home.page.scss'],
})
export class HomePage {
  public state: VaultServiceState;

  constructor(private vaultService: VaultService) {
    this.state = vaultService.state;
  }

  async setSession(data: string) {
    await this.vaultService.setSession(data);
  }

  async restoreSession() {
    await this.vaultService.restoreSession();
  }
}
```

:::note
Throughout the rest of this tutorial only new markup or required code will be provided. It is up to you to make sure that the correct imports and component definitions are added.
:::

Build the application and run it on an iOS and/or Android device. You should be able to enter some data and store it in the vault by clicking "Set Session Data." If you then shutdown the app and start it again, you should be able to restore it using "Restore Session Data."

## Locking and Unlocking the Vault

Now that we are storing data in the vault, it would be helpful to lock and unlock that data. The vault will automatically lock after `lockAfterBackgrounded` milliseconds of the application being in the background. We can also lock the vault manually if we so desire.

Add the following code to `vault.service.ts`:

```TypeScript
async lockVault() {
  await this.vault.lock();
}

async unlockVault() {
  await this.vault.unlock();
}
```

We can then add a couple of buttons to `home.page.html`:

```html
<ion-item>
  <ion-label>
    <ion-button expand="block" (click)="lockVault()">Lock Vault</ion-button>
  </ion-label>
</ion-item>

<ion-item>
  <ion-label>
    <ion-button expand="block" (click)="unlockVault()">Unlock Vault</ion-button>
  </ion-label>
</ion-item>
```

Then define their click events in `home.page.ts`:

```TypeScript
lockVault() {
  this.vaultService.lockVault();
}

unlockVault() {
  this.vaultService.unlockVault();
}
```

We can now lock and unlock the vault, though in our current state we cannot really tell. Our application should react in some way when the vault is locked. For example, we may want to clear specific data from memory. We may also wish to redirect to a page that will only allow the user to proceed if they unlock the vault.

In our case, we will just clear the `session` and have a flag that we can use to visually indicate if the vault is locked or not. We can do that by using the vault's `onLock` event.

First, we need to add a property to track whether the vault is locked or not. In `src/vault.service.ts`, add a boolean `isLocked` to the `VaultServiceState` definition:

```TypeScript
export interface VaultServiceState {
  state: string;
  isLocked: boolean;
}
```

And set it to `false` within `VaultService`:

```TypeScript
public state: VaultServiceState = {
  session: '',
  isLocked: false
}
```

Next, the vault's `onLock` and `onUnlock` events need to be handled. Modify the `init()` method:

```TypeScript
async init() {
    this.vault = Capacitor.getPlatform() === 'web'
      ? new BrowserVault(this.config)
      : new Vault(config);

    this.vault.onLock(() => {
      this.ngZone.run(() => {
        this.state.isLocked = true;
        this.state.session = undefined;
      });
    });

    this.vault.onUnlock(() => {
      this.ngZone.run(() => {
        this.state.isLocked = false;
      });
    });
  }
```

:::note
The events of Identity Vault are not aware of the change detection system of Angular. `ngZone.run()` ensures that the user interface updates on changes.
:::

---

**Note:** We use the `ngZone` `run` method to ensure our user interface updates on changes.

Then, update `home.page.html` to display `state.isLocked` value along with the session:

```html
<ion-item>
  <ion-label>
    <div>Session Data: {{ state.session }}</div>
    <div>Vault is locked: {{ state.isLocked }}</div>
  </ion-label>
</ion-item>
```

Build and run the application. When the user presses the "Lock Vault" button, the "Session Data" will be cleared out. Pressing "Unlock Vault" will cause "Vault is Locked" to show as false again. Pressing "Restore Session Data" will both unlock a vault and get the session data back.

In its current state, the vault is set to the `SecureStorage` vault type. We will be unable to observe any changes to the "Vault is Locked" value until we update the vault's configuration to a different vault type.

:::note
Vaults set to the `SecureStorage` vault type _do not_ lock. Data stored within this type of vault are still encrypted at rest and can be restored between sessions.
:::

In a few sections we will explore different vault types further, allowing us to test the ability to lock and unlock a vault. First, we will begin exploring the `Device` API.

## Device Level Capabilities

Identity Vault allows you to have multiple vaults within your application. However, there are some capabilities that Identity Vault allows you to control that are applicable to the device that the application is running on rather than being applicable to any given vault. For these items, we will use Identity Vault's `Device` API.

One such item is the "privacy screen." When an application is put into the background, the default behavior is for the OS to take a screenshot of the current page and display that as the user scrolls through the open applications. However, if your application displays sensitive information you may not want that information displayed at such a time. So, another option is to display the splash screen (on iOS) or a plain rectangle (on Android) instead of the screenshot. This is often referred to as a "privacy screen."

We will use the `Device.isHideScreenOnBackgroundEnabled()` method to determine if our application will currently display the privacy screen or not. Then we will use the `Device.setHideScreenOnBackground()` method to control whether it is displayed or not. Finally, we will hook that all up to a checkbox in the UI to allow the user to manipulate the value at run time.

We only want to interact with the Device API if we are actually running on a device, so we will also use Ionic's platform detection features to avoid using the Device API when running on the web. Our app is not targeting the web; we just want to ensure we can still use a web based development flow.

All of the following code applies to the `src/app/vault.service.ts` file.

First, import the `Device` API:

```TypeScript
import {
  ...
  Device
} from '@ionic-enterprise/identity-vault';
```

Next, add a property to track whether the privacy screen is locked or not. In `src/vault.service.ts`, add a boolean `isLocked` to the `VaultServiceState` definition:

```TypeScript
export interface VaultServiceState {
  state: string;
  isLocked: boolean;
}
```

And set it to `false` within `VaultService`:

```TypeScript
public state: VaultServiceState = {
  session: '',
  isLocked: false
}
```

Then, add the following code to the end of the `init()` method:

```TypeScript
this.state.privacyScreen =
  Capacitor.getPlatform() === 'web'
    ? Promise.resolve(false)
    : await Device.isHideScreenOnBackgroundEnabled();
```

Next, add a method to `VaultService`:

```TypeScript
setPrivacyScreen(enabled: boolean) {
  Device.setHideScreenOnBackground(enabled);
  this.state.privacyScreen = enabled;
}
```

Finally, we can add the checkbox to `home.page.html`:

```html
<ion-item>
  <ion-label>Use Privacy Screen</ion-label>
  <ion-checkbox
    [(ngModel)]="state.privacyScreen"
    (ionChange)="setPrivacyScreen()"
  ></ion-checkbox>
</ion-item>
```

Build the app and play around with changing the checkbox and putting the app in the background. In most applications, you would leave this value set by default. If you were going to change it, you would most likely do so on startup and leave it that way.

## Using Different Vault Types

The mechanism used to unlock a vault is determined by a combination of the `type` and the `deviceSecurityType` configuration settings.

The `type` setting can be set to any value from the `VaultType` enumeration:

- `SecureStorage`: Securely store the data in the keychain, but do not lock it.
- `DeviceSecurity`: When the vault is locked, it needs to be unlocked by a mechanism provided by the device.
- `CustomPasscode`: When the vault is locked, it needs to be unlocked via a custom method provided by the application. This is typically done in the form of a custom PIN dialog.
- `InMemory`: The data is never persisted. As a result, if the application is locked or restarted the data is gone.

If `VaultType.DeviceSecurity` is used, the optional `deviceSecurityType` setting can further refine the vault by assigning a value from the `DeviceSecurity` enumeration:

- `Biometrics`: Use the biometric authentication type specified by the device.
- `SystemPasscode`: Use the system passcode entry screen.
- `Both`: Use `Biometrics` with the `SystemPasscode` as a backup when `Biometrics` fails.

We specified `SecureStorage` when we set up the vault:

```typescript
const config: IdentityVaultConfig = {
  key: "io.ionic.getstartedivangular",
  type: VaultType.SecureStorage,
  deviceSecurityType: DeviceSecurityType.None,
  lockAfterBackgrounded: 2000,
  shouldClearVaultAfterTooManyFailedAttempts: true,
  customPasscodeInvalidUnlockAttempts: 2,
  unlockVaultOnLoad: false,
};
```

However, we can use the vault's `updateConfig()` method to change this at run time.

In our application, we don't want to use every possible combination. Rather than exposing the raw `type` and `deviceSecurityType` values to the rest of the application, let's define the types of authorization we _do_ want to support:

- `NoLocking`: We want to store the session data securely, but never lock it.
- `Biometrics`: We want to use the device's biometric mechanism to unlock the vault when it is locked.
- `SystemPasscode`: We want to use the device's passcode screen (typically a PIN or pattern) to unlock the vault when it is locked.

Now we have the types defined within the domain of our application. The only code within our application that will have to worry about what this means within the context of the Identity Vault configuration is our `VaultService`.

First, add another property to `VaultServiceState` in `src/vault.service.ts`:

```TypeScript
export interface VaultServiceState {
  session: string;
  isLocked: boolean;
  privacyScreen: boolean;
  lockType: 'NoLocking' | 'Biometrics' | 'SystemPasscode'
}
```

Next, set the initial state:

```TypeScript
  public state: VaultServiceState = {
    session: '',
    isLocked: false,
    privacyScreen: false,
    lockType: 'NoLocking'
  };
```

Then, define a method called `setLockType` within `VaultService`:

```TypeScript
setLockType() {
  let type: VaultType;
  let deviceSecurityType: DeviceSecurityType;

  switch (this.state.lockType) {
    case 'Biometrics':
      type = 'DeviceSecurity';
      deviceSecurityType = 'Biometrics';
      break;

    case 'SystemPasscode':
      type = 'DeviceSecurity';
      deviceSecurityType = 'SystemPasscode';
      break;

    default:
      type = 'SecureStorage';
      deviceSecurityType = 'SystemPasscode';
  }
  this.vault.updateConfig({ ...this.vault.config, type, deviceSecurityType });
}
```

Finally, add a group of radio button to `src/home/home.page.html`:

```html
<ion-item>
  <ion-radio-group [(ngModel)]="state.lockType" (ionChange)="setLockType()">
    <ion-list-header>
      <ion-label> Vault Locking Mechanism </ion-label>
    </ion-list-header>

    <ion-item>
      <ion-label>Do Not Lock</ion-label>
      <ion-radio value="NoLocking"></ion-radio>
    </ion-item>

    <ion-item>
      <ion-label>Use Biometrics</ion-label>
      <ion-radio
        [disabled]="!state.canUseBiometrics"
        value="Biometrics"
      ></ion-radio>
    </ion-item>

    <ion-item>
      <ion-label>Use System Passcode</ion-label>
      <ion-radio value="SystemPasscode"></ion-radio>
    </ion-item>
  </ion-radio-group>
</ion-item>
```

With the additional method `setLockType` added to `src/home/home.page.ts`:

```typescript
setLockType() {
  this.vaultService.setLockType();
}
```

Notice for the "Use Biometric" radio button, we are disabling it based on a `canUseBiometrics` value. We will need to code for that.

Add a property to the `VaultServiceState` interface:

```TypeScript
  canUseBiometrics: boolean;
}
```

Initialize `canUseBiometrics` in the `init` method of `VaultService`:

```TypeScript
    this.state.canUseBiometrics = await Device.isBiometricsEnabled();
```

Notice that we are using the `Device` API again here to determine if biometrics are both supported by the current device as well as enabled by the user. We don't want users to be able to choose that option unless the biometrics are properly set up on the device.

One final bit of housekeeping before building and running the application is that if you are using an iOS device you need to open the `Info.plist` file and add the `NSFaceIDUsageDescription` key with a value like "Use Face ID to access sensitive information."

Now when you run the app, you can choose a different locking mechanism and it should be used whenever you need to unlock the vault.

## Clear the Vault

One last method we will explore before we leave is the `clear()` method. The `clear()` API will remove all items from the vault and then remove the vault itself.

To show this in action, let's add a `vaultExists` property to `VaultServiceState`:

```TypeScript
export interface VaultServiceState {
  session: string;
  isLocked: boolean;
  privacyScreen: boolean;
  lockType: 'NoLocking' | 'Biometrics' | 'SystemPasscode';
  canUseBiometrics: boolean;
  vaultExists: boolean;
}
```

Let's then add a `clearVault()` function within `VaultService`. This function will call `vault.clear()`, reset the lockType to the default of `NoLocking`, and clear our session data cache.

```typescript
  async clearVault() {
    await this.vault.clear();
    this.state.lockType = 'NoLocking';
    this.state.session = undefined;
  }
```

In order to see whether the vault exists we need to create a method in `VaultService`:

```typescript
 async checkVaultExists(): Promise<void> {
    this.state.vaultExists = await this.vault.doesVaultExist();
  }
```

Lets call this method in the `init` method, in the `clearVault` method and in the `setSession` method:

```typescript
await this.checkVaultExists();
```

With that in place, open the `home.page.html` file and add a button to clear the vault by calling `clearVault()` on click:

```html
<ion-item>
  <ion-label>
    <ion-button expand="block" (click)="clearVault()">Clear Vault</ion-button>
  </ion-label>
</ion-item>
```

Add a `div` for displaying `vaultExists`:

```html
<div>Vault exists: {{ state.vaultExists }}</div>
```

Then in `home.page.ts`:

```typescript
  clearVault() {
    this.vaultService.clearVault();
  }
```

## Conclusion

This walk-through has implemented using Identity Vault in a very manual manner, allowing for a lot of user interaction with the vault. In an real application, functionality would instead be a part of several programmatic workflows.

At this point, you should have a good idea of how Identity Vault works. There is still more functionality that can be implemented. Be sure to check out our HowTo documents to determine how to facilitate specific areas of functionality within your application.
