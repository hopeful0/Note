# original
# links=https://github.com/rovo89/XposedBridge/wiki
# cdate=2016/02/15
# ldate=2016/02/15

Home

Development tutorial
	rovo89 edited this page on 13 Aug 2014 · 11 revisions 
──────────────────────────────────────────────────────────────────────────────
Alright.. you want to learn how you can create a new module for Xposed? Then read this tutorial (or let's rather call it "extensive essay") and learn how to approach this. This includes not only the technical "create this file and insert ...", but also the thinking behind it, which is the step where value is created and for which you really need to understand what you do and why. If you feel like "TL;DR", you can just look at the final source code and read the "Making the project an Xposed module" chapter. But you will get a better understanding if you read the whole tutorial. You will save the time spent for reading this later because you don't have to figure everything out yourself.
The modification subject
──────────────────────────────────────────────────────────────────────────────
You will recreate the red clock example that can be found at Github as well. It includes changing the color of the status bar clock to red and adding a smiley. I'm choosing this example because it is a rather small, but easily visible change. Also, it uses some of the basic methods provided by the framework.
How Xposed works
──────────────────────────────────────────────────────────────────────────────
Before beginning with your modification, you should get a rough idea how Xposed works (you might skip this section though if you feel too bored). Here is how:

There is a process that is called "Zygote". This is the heart of the Android runtime. Every application is started as a copy ("fork") of it. This process is started by an /init.rc script when the phone is booted. The process start is done with /system/bin/app_process, which loads the needed classes and invokes the initialization methods.

This is where Xposed comes into play. When you install the framework, an extended app_process executable is copied to /system/bin. This extended startup process adds an additional jar to the classpath and calls methods from there at certain places. For instance, just after the VM has been created, even before the main method of Zygote has been called. And inside that method, we are part of Zygote and can act in its context.

The jar is located at /data/data/de.robv.android.xposed.installer/bin/XposedBridge.jar and its source code can be found here. Looking at the class XposedBridge, you can see the main method. This is what I wrote about above, this gets called in the very beginning of the process. Some initializations are done there and also the modules are loaded (I will come back to module loading later).
Method hooking/replacing

What really creates the power of Xposed is the possibility to "hook" method calls. When you do a modification by decompiling an APK, you can insert/change commands directly wherever you want. However, you will need to recompile/sign the APK afterwards and you can only distribute the whole package. With the hooks you can place with Xposed, you can't modify the code inside methods (it would be impossible to define clearly what kind of changes you want to do in which place). Instead, you can inject your own code before and after methods, which are the smallest unit in Java that can be addressed clearly.

XposedBridge has a private, native method hookMethodNative. This method is implemented in the extended app_process as well. It will change the method type to "native" and link the method implementation to its own native, generic method. That means that every time the hooked method is called, the generic method will be called instead without the caller knowing about it. In this method, the method handleHookedMethod in XposedBridge is called, passing over the arguments to the method call, the this reference etc. And this method then takes care of invoking callbacks that have been registered for this method call. Those can change the arguments for the call, change instance/static variables, invoke other methods, do something with the result... or skip anything of that. It is very flexible.

Ok, enough theory. Let's create a module now!

Creating the project
──────────────────────────────────────────────────────────────────────────────
A module is normal app, just with some special meta data and files. So begin with creating a new Android project. I assume you have already done this before. If not, the official documentation is quite detailed. When asked for the SDK, I chose 4.0.3 (API 15). I suggest you try this as well and do not start experiments yet. You don't need to create an activity because the modification does not have any user-interface. After answering that question, you should have a blank project.

Making the project an Xposed module
──────────────────────────────────────────────────────────────────────────────
Now let's turn the project into something loaded by Xposed, a module. Several steps are required for this.
AndroidManifest.xml

The module list in the Xposed Installer looks for applications with a special meta data flag. You can create it by going to AndroidManifest.xml => Application => Application Nodes (at the bottom) => Add => Meta Data. The name should be xposedmodule and the value true. Leave the resource empty. Then repeat the same for xposedminversion (see below) and xposeddescription (a very short description of your module). The XML source will now look like this:

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="de.robv.android.xposed.mods.tutorial"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk android:minSdkVersion="15" />

    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" >
        <meta-data
            android:name="xposedmodule"
            android:value="true" />
        <meta-data
            android:name="xposeddescription"
            android:value="Easy example which makes the status bar clock red and adds a smiley" />
        <meta-data
            android:name="xposedminversion"
            android:value="30" />
    </application>
</manifest>
XposedBridgeApi.jar

Next, make the XposedBridge API known to the project. You can download XposedBridgeApi-<version>.jar from the first post of this XDA thread. Copy it into a subfolder called lib. Then right-click on it and select Build Path => Add to Build Path. The <version> from the file name is the one you insert as xposedminversion in the manifest.

    Make sure that the API classes are not included (but only referenced) in your compiled APK, otherwise you will get an IllegalAccessError. Files in the libs (with "s") folder are automatically included by Eclipse, so don't put the API file there.

Module implementation

Now you can create a class for your module. Mine is named "Tutorial" and is in the package de.robv.android.xposed.mods.tutorial:

package de.robv.android.xposed.mods.tutorial;

public class Tutorial {

}

For the first step, we will just do some logging to show that the module was loaded. A module can have a few entry points. Which one(s) you choose depends on the what you want to modify. You can have Xposed call a function in your module when the Android system boots, when a new app is about to be loaded, when the resources for an app are initialised and so on.

A bit further down this tutorial, you will learn that the necessary changes need to be done in one specific app, so let's go with the "let me know when a new app is loaded" entry point. All entry points are marked with a sub-interface of IXposedMod. In this case, it's IXposedHookLoadPackage which you need to implement. It's actually just one method with one parameter that gives more information about the context to the implementing module. In our example, let's log the name of the loaded app:
package de.robv.android.xposed.mods.tutorial;

import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XposedBridge;
import de.robv.android.xposed.callbacks.XC_LoadPackage.LoadPackageParam;

public class Tutorial implements IXposedHookLoadPackage {
    public void handleLoadPackage(final LoadPackageParam lpparam) throws Throwable {
        XposedBridge.log("Loaded app: " + lpparam.packageName);
    }
}

This log method writes the message to the standard logcat (tag Xposed) and to /data/data/de.robv.android.xposed.installer/log/debug.log` (which is easily accessible via the Xposed Installer).
assets/xposed_init

The only thing that is still missing now is a hint for XposedBridge which classes contain such entry points. This is done via a file called xposed_init. Create a new text file with that name in the assets folder. In this file, each line contains one fully qualified class name. In this case, this is de.robv.android.xposed.mods.tutorial.Tutorial.
Trying it out
──────────────────────────────────────────────────────────────────────────────
Save your files. Then run your project as Android application. As this is the first time you install it, you need to enable it before you can use it. Open the Xposed Installer app and make sure you have installed the framework. Then go to the "Modules" tab. You should find your app in there. Check the box to enable it. Then reboot. You will not see a difference of course, but if you check the log, you should see something like this:

Loading Xposed (for Zygote)...
Loading modules from /data/app/de.robv.android.xposed.mods.tutorial-1.apk
  Loading class de.robv.android.xposed.mods.tutorial.Tutorial
Loaded app: com.android.systemui
Loaded app: com.android.settings
... (many more apps follow)

Voilà! That worked. You now have an Xposed module. It could just be a bit more useful than writing logs...
Exploring your target and finding a way to modify it
──────────────────────────────────────────────────────────────────────────────
Ok, so now begins the part that can be very different depending on what you want to do. If you have modded APKs before, you probably know how to think here. In general, you first need to get some details about the implementation of the target. In this tutorial, the target is the clock in the statusbar. It helps to know that the statusbar and lots of other things are part of the SystemUI. So let's begin our search there.

Possibility one: Decompile it. This will give you the exact implementation, but it is hard to read and understand because you get smali format. Possibility two: Get the AOSP sources (e.g. here or here and look there. This can be quite different from your ROM, but in this case it is a similar or even the same implementation. I would look at AOSP first and see if that is enough. If I need more details, look at the actual decompiled code.

You can look for classes with "clock" in their name or containing that string. Other things to look for are resources and layout used. If you downloaded the official AOSP code, you can start looking in frameworks/base/packages/SystemUI. You will find quite a few places where "clock" appears. This is normal and indeed there will be different ways to implement a modification. Keep in mind that you can "only" hook methods. So you have to find a place where you can insert some code to do the magic either before, after or replacing a method. You should hook methods that are as specific as possible, not ones that are called thousands of times to avoid performance issues and unintended side-effects.

In this case, you might find that the layout res/layout/status_bar.xml contains a reference to a custom view with the class com.android.systemui.statusbar.policy.Clock. Multiple ideas might come to your mind now. The text color is defined via a textAppearance attribute, so the cleanest way to change it would be to change the appearance definition. However, the this is not possible to change styles with the Xposed framework and probably won't be (it's too deep in native code). Replacing the layout for the statusbar would be possible, but an overkill for the small change you are trying to make. Instead, look at this class. There is a method called updateClock, which seems to be called every minute to update the time:

final void updateClock() {
    mCalendar.setTimeInMillis(System.currentTimeMillis());
    setText(getSmallTime());
}

That looks perfect for modifications because it is a very specific method which seems to be the only means of setting the text for the clock. If we add something after every call to this method that changes the color and the text of the clock, that should work. So let's do it.

    For the text color alone, there is an even better way. See the example for "Modifying layouts" on "Replacing resources".
Using reflection to find and hook a method
──────────────────────────────────────────────────────────────────────────────
What do we already know? We have a method updateClock in class com.android.systemui.statusbar.policy.Clock that we want to intercept. We found this class inside the SystemUI sources, so it will only be available in the process for the SystemUI. Some other classes belong to the framework and are available everywhere. If we tried to get any information and references to this class directly in the handleLoadPackage mehod, this would fail because it is the wrong process. So let's implement a condition to execute certain code only when a specified package is about to be loaded:

public void handleLoadPackage(LoadPackageParam lpparam) throws Throwable {
    if (!lpparam.packageName.equals("com.android.systemui"))
        return;

    XposedBridge.log("we are in SystemUI!");
}

Using the parameter, we can easily check if we are in the correct package. Once we verified that, we get access to the classes in that packages with the ClassLoader which is also referenced from this variable. Now we can look for the com.android.systemui.statusbar.policy.Clock class and its updateClock method and tell XposedBridge to hook it:

package de.robv.android.xposed.mods.tutorial;

import static de.robv.android.xposed.XposedHelpers.findAndHookMethod;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.callbacks.XC_LoadPackage.LoadPackageParam;

public class Tutorial implements IXposedHookLoadPackage {
    public void handleLoadPackage(final LoadPackageParam lpparam) throws Throwable {
        if (!lpparam.packageName.equals("com.android.systemui"))
            return;

        findAndHookMethod("com.android.systemui.statusbar.policy.Clock", lpparam.classLoader, "updateClock", new XC_MethodHook() {
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                // this will be called before the clock was updated by the original method
            }
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                // this will be called after the clock was updated by the original method
            }
    });
    }
}

findAndHookMethod is a helper function. Note the static import, which is automatically added if you configure it as described in the linked page. This method looks up the Clock class using the ClassLoader for the SystemUI package. Then it looks for the updateClock method in it. If there were any parameters to this method, you would have to list the types (classes) of these parameters afterwards. There are different ways to do this, but as our method doesn't have any parameters, let's skip this for now. As the last argument, you need to provide an implementation of the XC_MethodHook class. For smaller modifications, you can use a anonymous class. If you have much code, it's better to create a normal class and only create the instance here. The helper will then do everything necessary to hook the method as described above.

There are two methods in XC_MethodHook that you can override. You can override both or even none, but the latter makes absolutely no sense. These methods are beforeHookedMethod and afterHookedMethod. It's not too hard to guess that the are executed before/after the original method. You can use the "before" method to evaluate/manipulate the parameters of the method call (via param.args) and even prevent the call to the original method (sending your own result). The "after" method can be used to do something based on the result of the original method. You can also manipulate the result at this point. And of course, you can add your own code which should be executed exactly before/after the method call.

    If you want to replace a method completely, have a look at the subclass XC_MethodReplacement instead, where you just need to override replaceHookedMethod.

XposedBridge keeps a list of registered callbacks for each hooked method. Those with highest priority (as defined in hookMethod) are called first. The original method has always the lowest priority. So if you have hooked a method with callbacks A (prio high) and B (prio default), then whenever the hooked method is called, the control flow will be this: A.before -> B.before -> original method -> B.after -> A.after. So A could influence the arguments B gets to see, which could further change them before passing them on. The result of the original method can be processed by B first, but A has the final word what the original caller gets.
Final steps: Execute your own code before/after the method call
──────────────────────────────────────────────────────────────────────────────
Alright, you have now a method that is called every time the updateClock method is called, with exactly that context (i.e. you're in the SystemUI process). Now let's modify something.

First thing to check: Do we have a reference to the concrete Clock object? Yes we have, it's in the param.thisObject parameter. So if the method was called with myClock.updateClock(), then param.thisObject would be myClock.

Next: What can we do with the clock? The Clock class is not available, you can't cast param.thisObject to class (don't even try to). However it inherits from TextView. So you can use methods like setText, getText and setTextColor once you have casted the Clock reference to TextView. The changes should be done after the original method has set the new time. As there is nothing to do before the method call, we can leave out the beforeHookedMethod. Calling the (empty) "super" method is not necessary.

So here is the complete source code:

package de.robv.android.xposed.mods.tutorial;

import static de.robv.android.xposed.XposedHelpers.findAndHookMethod;
import android.graphics.Color;
import android.widget.TextView;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.callbacks.XC_LoadPackage.LoadPackageParam;

public class Tutorial implements IXposedHookLoadPackage {
    public void handleLoadPackage(final LoadPackageParam lpparam) throws Throwable {
        if (!lpparam.packageName.equals("com.android.systemui"))
            return;

        findAndHookMethod("com.android.systemui.statusbar.policy.Clock", lpparam.classLoader, "updateClock", new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                TextView tv = (TextView) param.thisObject;
                String text = tv.getText().toString();
                tv.setText(text + " :)");
                tv.setTextColor(Color.RED);
            }
        });
    }
}
 Be happy about the result
──────────────────────────────────────────────────────────────────────────────
Now install/start your app again. As you have already enabled it in the Xposed Installer when you started it the first time, you do not need to do that again, rebooting is enough. However, you will want to disable the red clock example if you were using it. Both use the default priority for their updateClock handler, so you cannot know which one will win (it actually depends on the string representation of the handler method, but don't rely on that).
Conclusion
──────────────────────────────────────────────────────────────────────────────
I know that this tutorial was very long. But I hope you can now not only implement a green clock, but also something completely different. Finding good methods to hook is a matter of experience, so start with something rather easy. Try using the log functions a lot in the beginning to make sure that everything is called when you expect it. And now: Have fun!
──────────────────────────────────────────────────────────────────────────────
Helper
	rovo89 edited this page on 29 Jan 2014 · 4 revisions 
──────────────────────────────────────────────────────────────────────────────
There are many helper methods in Xposed that can make developing a module much easier.
Class XposedBridge
──────────────────────────────────────────────────────────────────────────────
log

The log method is an easy way of logging debug output to the standard logcat and a file called /data/xposed/debug.log. It can take the log message or a Throwable. In the latter case, it will print the stack trace.
hookAllMethods / hookAllConstructors

You can use these methods if you want to hook all methods with a specific name or all constructors in a class. This is useful if there are different variants, but you want to execute some code before/after any of them has been called. Keep in mind that other ROMs might have additional variants that will also be hooked by this. Especially, be careful about the args you get in the callback.
Class XposedHelpers
──────────────────────────────────────────────────────────────────────────────
I recommend adding this class to the Eclipse favorites for static imports: Window => Preferences => Java => Editor => Content Assist => Favorites => New Type, enter de.robv.android.xposed.XposedHelpers. By doing this, Eclipse will automatically suggest the methods from this class if you start typing "get.." for example and will create a static import of that method (that means you don't have the classname visible in your code).
findMethod / findConstructor / findField

There are a couple of methods for retrieving methods, constructors and fields without using reflection yourself. Also, you can find methods and constructors using a "best match" for certain parameter types. So for example, you can call findMethodBestMatch(Class<?> clazz, String methodName, Object... args) with a TextView argument and it will also find a method with the given name that takes a View parameter if no more specific variant exists.
callMethod / callStaticMethod / newInstance

Making use of the findXXX methods mentioned above, these methods make it easy to call a method or create a new instance of a class. The caller doesn't have to use reflection for this. There is no need to retrieve the method before, just use these methods to call it on-the-fly. The types of the parameters are automatically copied from the actual parameter values and the best-matching method is called. In case you want to explicitly specify the type for a parameter, create a Class<?> array and pass it to callXXX/newInstance. You can leave some of the items in the array empty (null) to use the class of the actual parameter, but the array length has to match the number of parameters.
getXXXField / setXXXField / getStaticXXXField /setStaticXXXField

These are wrappers to easily get and set the content of instance and class variables. You just need the reference to the object, the field name and type (and the new value for setters of course). If you want to get/set a static field and don't have an object reference, you can use the getStaticXXX and setStaticXXX methods. There is however no need to differentiate between static and instance fields when you have an object reference, the getXXX and setXXX can set both.
getAdditionalXXXField / setAdditionalXXXField

These methods let you associate any values with either an instance of an object or a whole class (like a static field). The values are stored in a key-value-map, so you can save multiple values per object. The key can be any string, including names of fields that the object actually has. Please note that it you cannot retrieve a value you stored with setAdditionalStaticField by calling getAdditionalInstanceField. Use getAdditionalStaticField instead, which has a variant which takes an object and looks up its class automatically.
assetAsByteArray

This method returns an asset as a byte array. If you want to load your module's resources, you can use something like this:

public class XposedTweakbox implements IXposedHookZygoteInit {
    @Override
    public void initZygote(StartupParam startupParam) throws Throwable {
        Resources tweakboxRes = XModuleResources.createInstance(startupParam.modulePath, null);
        byte[] crtPatch = assetAsByteArray(tweakboxRes, "crtfix_samsung_d506192d5049a4042fb84c0265edfe42.bsdiff");
...

getMD5Sum

Returns the MD5 sum of a file on the file system. The current app needs read access to the file (in the init method you have root permissions, so that should not be a problem).
getProcessPid

Finds a process by the first part of its /proc/[pid]/cmdline and returns its PID as a String.

──────────────────────────────────────────────────────────────────────────────
Replacing resources
──────────────────────────────────────────────────────────────────────────────
Xposed makes it easy to replace resources, for example images or strings. Here is how:
Simple resources
──────────────────────────────────────────────────────────────────────────────
@Override
public void initZygote(IXposedHookZygoteInit.StartupParam startupParam) throws Throwable {
    XResources.setSystemWideReplacement("android", "bool", "config_unplugTurnsOnScreen", false);
}

@Override
public void handleInitPackageResources(InitPackageResourcesParam resparam) throws Throwable {
    // replacements only for SystemUI
    if (!resparam.packageName.equals("com.android.systemui"))
        return;

    // different ways to specify the resources to be replaced
    resparam.res.setReplacement(0x7f080083, "YEAH!"); // WLAN toggle text. You should not do this because the id is not fixed. Only for framework resources, you could use android.R.string.something
    resparam.res.setReplacement("com.android.systemui:string/quickpanel_bluetooth_text", "WOO!");
    resparam.res.setReplacement("com.android.systemui", "string", "quickpanel_gps_text", "HOO!");
    resparam.res.setReplacement("com.android.systemui", "integer", "config_maxLevelOfSignalStrengthIndicator", 6);
}

This is for "simple" replacements where you can give the replacement value directly. This works for: Boolean, Color, Integer, int[], String and String[].

As you can see, there are a few different ways to set a replacement resource. For those resources that are part of the Android framework (available to all apps) and that should be replaced everywhere, you call XResources.setSystemWideReplacement(...) in the initZygote method. For app-specific resources, you need to call res.setReplacement in hookInitPackageResources after verifying that you are in the correct app. You should not use setSystemWideReplacement at this time as it might have side-effects you didn't expect.

Replacing Drawables works similar. However, you can't just use a Drawable as replacement because this might result in the same instance of the Drawable being referenced by different ImageViews. Therefore, you need to use a wrapper:

resparam.res.setReplacement("com.android.systemui", "drawable", "status_bar_background", new XResources.DrawableLoader() {
    @Override
    public Drawable newDrawable(XResources res, int id) throws Throwable {
        return new ColorDrawable(Color.WHITE);
    }
});

Complex resources
──────────────────────────────────────────────────────────────────────────────
More complex resources (like animated Drawables) have to be referenced from your module's resources. Let's say you want to replace the battery icon. Here is the code:

package de.robv.android.xposed.mods.coloredcirclebattery;

import android.content.res.XModuleResources;
import de.robv.android.xposed.IXposedHookInitPackageResources;
import de.robv.android.xposed.IXposedHookZygoteInit;
import de.robv.android.xposed.callbacks.XC_InitPackageResources.InitPackageResourcesParam;

public class ColoredCircleBattery implements IXposedHookZygoteInit, IXposedHookInitPackageResources {
    private static String MODULE_PATH = null;

    @Override
    public void initZygote(StartupParam startupParam) throws Throwable {
        MODULE_PATH = startupParam.modulePath;
    }

    @Override
    public void handleInitPackageResources(InitPackageResourcesParam resparam) throws Throwable {
        if (!resparam.packageName.equals("com.android.systemui"))
            return;

        XModuleResources modRes = XModuleResources.createInstance(MODULE_PATH, resparam.res);
        resparam.res.setReplacement("com.android.systemui", "drawable", "stat_sys_battery", modRes.fwd(R.drawable.battery_icon));
        resparam.res.setReplacement("com.android.systemui", "drawable", "stat_sys_battery_charge", modRes.fwd(R.drawable.battery_icon_charge));
    }
}

    You can name your replacement resources as you like. I chose battery_icon over stat_sys_battery to make them easier to distinguish in this text.

Then you add the Drawables "battery_icon" and "battery_icon_charge" to your module. In the easiest case, this means you add "res/drawables/battery_icon.png" and "res/drawables/battery_icon_charge.png", but you can use all the ways Android provides to define resources. So for animated icons, you would use XML files with an animation-list and references to other Drawables, which of course have to be in your module as well.

With these replacements, you ask Xposed to forward all requests for a certain resource to your own module. So whatever you can do with resources in your own app should work as a replacement as well. This also means that you can make use of qualifiers, for example if you need different resources for landscape or lower densities. Translations could be provided the same way. Also you will probably need this if the original resource uses qualifiers. You can't replace only the Spanish version of a text. As mentioned, the request is forwarded, so it's completely handled by your module's resources, which doesn't know that other translations exists.

This technique should work with basically all the resource type, except for special things like themes.
Modifying layouts
──────────────────────────────────────────────────────────────────────────────
Although you could theoretically replace layouts completely with the technique described above, this has many downsides. You have to copy the complete layout from the original, which reduces compatibility with other ROMs. Themes might be lost. Only one module can replace a layout. If two modules try it, the last one will win. Most important, IDs and references to other resources are pretty hard to define. Therefore, I really don't recommend this.

As a good alternative, you can use post-inflate hooks. This is what you do:

@Override
public void handleInitPackageResources(InitPackageResourcesParam resparam) throws Throwable {
    if (!resparam.packageName.equals("com.android.systemui"))
        return;

    resparam.res.hookLayout("com.android.systemui", "layout", "status_bar", new XC_LayoutInflated() {
        @Override
        public void handleLayoutInflated(LayoutInflatedParam liparam) throws Throwable {
            TextView clock = (TextView) liparam.view.findViewById(
                    liparam.res.getIdentifier("clock", "id", "com.android.systemui"));
            clock.setTextColor(Color.RED);
        }
    }); 
}

The callback handleLayoutInflated is called whenever the layout "status_bar" has been inflated. Inside the LayoutInflatedParam object that you get as a parameter, you can find the just created view and can modify it as needed. You also get resNames to identify which layout the method was called for (in case you are using the same method for multiple layouts) and variant, which might for example contain layout-land if that it is the version of the layout that was loaded. res helps you to get IDs or additonal resources from the same source as the layout.

──────────────────────────────────────────────────────────────────────────────
