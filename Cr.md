# Xamarin DependencyService
***

|               | Document Details 
|:------------- |:----------------
| Owner:        | Propelics       
| Organization: | Anexinet        
| File created: | 01-07-2018      
| Last modified:| 03-07-2018      
| Subject:      | Xamarin DependencyService      

***

## 1.- Introduction

**DependencyService** allows apps to call into platform-specific functionality from shared code. This functionality enables Xamarin.Forms apps to do anything that a native app can do. DependencySercive is a dependency resolver. In practice, an interface is defined and DependencyService finds the correct implementation of that interface from the various platform projects.

## 2.- How DependencyService Works

Xamarin.Forms apps need four components to use DependencyService:

* **Interface:** The required functionality is defined by an interface in shared code this include methods and properties.
* **Implementation Per Platform:** Classes that implement the interface must be added to each platform project.
* **Registration:**  Each implementing class must be registered with DependencyService via a metadata attribute. Registration enables DependencyService to find the implementing class and supply it in place of the interface at runtime.
* **Call to DependencyService:** Shared code needs to explicitly call DependencyService to ask for implementations of the interface.

>Note that implementations must be provided for each platform project in your solution. Platform projects without implementations will fail at runtime.

##  3.- DependencyService Components 

### **Interface**
The interface you design will define how you interact with platform-specific functionality. It will be only a contract for exposing standard functionalities. The example below specifies a simple interface for obtaining the device orientation.

    public enum DeviceOrientations
    {
        Undefined,
        Landscape,
        Portrait
    }

    public interface IDeviceOrientation
    {
        DeviceOrientations GetOrientation();
    }

### **Implementation Per Platfom**
Once a suitable interface has been designed, that interface must be implemented in the project for each platform that you are targeting.
The following code demonstrates the implementation of the created interface in IOS:

    public class DeviceOrientationImplementation : IDeviceOrientation
    {
        public DeviceOrientationImplementation()
        {
        }

        public DeviceOrientations GetOrientation()
        {
            var currentOrientation = UIApplication.SharedApplication.StatusBarOrientation;
            bool isPortrait = currentOrientation == UIInterfaceOrientation.Portrait
                || currentOrientation == UIInterfaceOrientation.PortraitUpsideDown;

            return isPortrait ? DeviceOrientations.Portrait : DeviceOrientations.Landscape;
        }
    }

The following code demostrates the implementation of the created interface in Android:

    public class DeviceOrientationImplementation : IDeviceOrientation
    {
        public DeviceOrientationImplementation()
        {
        }

        public static void Init() { }

        public DeviceOrientations GetOrientation()
        {
            IWindowManager windowManager = Android.App.Application.Context.GetSystemService(Context.WindowService).JavaCast<IWindowManager>();

            var rotation = windowManager.DefaultDisplay.Rotation;
            bool isLandscape = rotation == SurfaceOrientation.Rotation90 || rotation == SurfaceOrientation.Rotation270;
            return isLandscape ? DeviceOrientations.Landscape : DeviceOrientations.Portrait;
        }
    }
    
### **Registration**
Each implementation of the interface needs to be registered with DependencyService with a metadata attribute.
The following code shows the registration in both platforms:

    [assembly: Xamarin.Forms.Dependency(typeof(DeviceOrientationImplementation))]
    namespace testApp.Droid
    {
        public class DeviceOrientationImplementation : IDeviceOrientation
        { ...

### **Call to DependencyService**
Once the project has been set up with a common interface and implementations for each platform, use DependencyService to get the right implementation at runtime.

    var orientation = DependencyService.Get<IDeviceOrientation>().GetOrientation();

DependencyService.Get<T> will find the correct implementation of interface T.

##  4.- DependencyService Summary
The DependencyService allow us to access the native functionalities from each platform resolving the implementation in a simple interface. With this technique, specific API from native targets will be available to be used at our shared code whenever is needed.

##  5.- Reference
You can find more detailed information along with few more examples at the official Microsoft Xamarin documentation & samples web sites:

* Documentation:
<https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/dependency-service/introduction>

* Additional Sample:
<https://developer.xamarin.com/samples/xamarin-forms/UsingDependencyService/>
