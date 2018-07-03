# Xamarin Custom Renderers
***

|               | Document Details 
|:------------- |:----------------
| Owner:        | Propelics       
| Organization: | Anexinet        
| File created: | 01-07-2018      
| Last modified:| 01-07-2018      
| Subject:      | Xamarin Custom Renderers      

***

## 1.- Introduction

**Custom Renderers** provide a powerful approach for customizing the appearance and behavior of Xamarin.Forms controls. They can be used for small styling changes or sophisticated platform-specific layout and behavior customization.

Developers can implement their own custom **Custom Renderers** classes to customize the appearance and/or behavior of a control. Custom renderers for a given type can be added to one application project to customize the control in one place while allowing the default behavior on other platforms; or different custom renderers can be added to each application project to create a different look and feel on iOS, Android, and the Universal Windows Platform (UWP). However, implementing a custom renderer class to perform a simple control customization is often a heavy-weight response. Effects simplify this process, and are typically used for small styling changes.

## 2.- How Custom Renderers are created

The followings steps illustartes how a custom control is created under Xamarin.Forms:

1. First, you need to create a folder inside your shared code named **Controls**. This new folder will contain the controls we will customize later. Then, create a class following the pattern: ```Extended + [controlName] + .cs``` e.g. ExtendedLabel.cs, ExtendedButton.cs, etc.
2. After creating the class you will need to make it inherit from the base control you want to extend. e.g. Label, Entry, Button.
3. Develop your class logic following Propelics standards. Here you will be creating **Bindable Properties**, **Event Handlers** and **Methods** that you will later use once you create the platform specific class implementations.
4. Create a folder in each specif platform you want to extend the control, name the folder **Renderers**.
5. Create a class inside the created folder and name this class according to Propelics Standards: ```Extended + [controlName] + Renderer + .cs``` e.g. ExtendedLabelRenderer.cs, ExtendedEntryRenderer.cs. Once the class is created, the required assembly tag needs to be added to the top of the class in order to make it available for the rest of the code. ```[assembly: ExportRenderer(typeof(Type), typeof(TypeRenderer))]``` Finally, you need to inherit the class from the corresponding **Renderer Type**. You can find in the following URL a list of existing renderers and their corresponding classes:
<https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/custom-renderer/renderers>
6. After the previos steps, now you must override the method: **OnElementChanged** of the current class.
7. Assign the instance of the base control in a variable.
8. Implement the actual control customization based on properties and methods created in the Xamarin.Forms control class. This process of customization **should** be implemented in separated methods in order to improve the readability of the control. The created methods will be executed inside the **OnElementChanged** method.
9. Add parameter validation according to the renderer base class doc.
10. The custom control is ready to be consumed from the **Core** project.

>Note: Every Xamarin.Forms control has an accompanying renderer for each platform that creates an instance of a native control.

##  3.- Custom Renderers in Detail

### **Class inside the shared code**
The following code demostrates how to create the the class that will extend from the base control explained in steps **1 to 3**.

```c#
    // Class extends from base control class
    public class ExtendedEntry : Entry
    {
        // Bindable Properties
    	public static readonly BindableProperty SetPaddingLeftProperty = BindableProperty.Create(nameof(PaddingLeftProperty), typeof(int), typeof(ExtendedEntry), 0);
    	public static readonly BindableProperty SetMarginBottomProperty = BindableProperty.Create("SetMarginBottom", typeof(float), typeof(ExtendedEntry), 0.0f);
    	
    	//Properties Implementation
        public int PaddingLeftProperty
        {
            get
            {
                return (int)GetValue(SetPaddingLeftProperty);
            }
            set
            {
                SetValue(SetPaddingLeftProperty, value);
            }
        }

        public float MarginBottom
        {
            get
            {
                return (float)GetValue(SetMarginBottomProperty);
            }
            set
            {
                SetValue(SetMarginBottomProperty, value);
            }
        }
    }
```

### **Implementation Per Platform**
The following code demonstrates the implementation of the created class at platform specific level, in this case **Androird**:

```c#
    [assembly: ExportRenderer(typeof(ExtendedEntry), typeof(ExtendedEntryRenderer))]
    namespace App.Droid
    {
	    public class ExtendedEntryRenderer : EntryRenderer
	    {
	        private ExtendedEntry _extendedEntry;
	
	        public ExtendedEntryRenderer(Context context) : base(context)
	        {
	        }
	
	        protected override void OnElementChanged(ElementChangedEventArgs<Entry> e)
	        {
	            base.OnElementChanged(e);
	
	            if (e.OldElement == null)
	            {
	            	// Place here the call to your customization methods.
	                SetPaddingLeftProperty();
	                SetMarginBottomProperty();
	            }
	        }
	        
	        // Customize your control here        
	        private void SetPaddingLeftProperty()
	        {
	            if (_extendedEntry.PaddingLeftProperty != 0)
	            {
	                this.Control.SetPadding(_extendedEntry.PaddingLeftProperty, 0, 0, 0);
	            }
	        }
	        
	        private void SetMarginBottomProperty()
	        {
                int pixelBottom = (int)TypedValue.ApplyDimension(ComplexUnitType.Dip, 10, Context.Resources.DisplayMetrics);

                if (_extendedEntry.isBottomMarginEnabled)
                {
                    pixelBottom = (int)TypedValue.ApplyDimension(ComplexUnitType.Dip, _extendedEntry.MarginBottom, Context.Resources.DisplayMetrics);
                }
                
                Control.SetPadding(0, 0, 0, pixelBottom);
	        }
        
	    }
    }
```

The following code demonstrates the implementation of the created class at platform specific level, in this case **IOS**:

```c#
    [assembly: ExportRenderer(typeof(ExtendedEntry), typeof(ExtendedEntryRenderer))]
    namespace App.iOS
    {
	    public class ExtendedEntryRenderer : EntryRenderer
	    {
	         private ExtendedEntry _extendedEntry;
			 private UIEdgeInsets EdgeInsets { get; set; }
			 
	        public ExtendedEntryRenderer(Context context) : base(context)
	        {
	        }
	
	        protected override void OnElementChanged(ElementChangedEventArgs<Entry> e)
	        {
	            base.OnElementChanged(e);
	
	            if (e.OldElement == null)
	            {
	            	// Place here the call to your customization methods.
	                SetPaddingLeftProperty();
	                SetMarginBottomProperty();
	            }
	        }
	        
	        // Customize your control here        
	        private void SetPaddingLeftProperty()
	        {
	            if (_extendedEntry.PaddingLeftProperty != 0)
	            {
	               Control.LeftView = new UIView(new CGRect(0,0,_extendedEntry.PaddingLeftProperty,0));
                   Control.LeftViewMode = UITextFieldViewMode.Always;
	            }
	        }
	        private void SetMarginBottomProperty()
	        {
                if (_extendedEntry.isBottomMarginEnabled)
	            {
	                UIHelper.SetMargin(Control, _extendedEntry.isBottomMarginEnabled, _extendedEntry.MarginBottom);
	            }
	        }
        
	    }
    }
```   
    
### **Registration**
Each implementation of the interface needs to be registered with a metadata attribute.
The following code shows the registration in both platforms:

```c#
    [assembly: ExportRenderer(typeof(ExtendedEntry), typeof(ExtendedEntryRenderer))]
```

### **XAML Call to Custom Renderers**
In order to utilize the created control over your XAML views you need to follow the two next steps:

* First, you need to include the namespace of your Custom Renderers folder into the view you want to use the control: ```xmlns:customControls="clr-namespace:App.Views.CustomRenders"``` See next example:

```xml#
        <?xml version="1.0" encoding="UTF-8"?>
        <local:BasePage xmlns="http://xamarin.com/schemas/2014/forms"
            xmlns:local="clr-namespace:App.Views"
            xmlns:customControls="clr-namespace:App.Views.CustomRenders"
            xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
            x:Class="App.Views.SamplePage">
``` 
           
* Then, you need to add the corresponding XAML tag where you want to use the Custom Control. After properly importing it in the previous step, this will be available to be used in the view.

```html#
        <StackLayout HorizontalOptions="EndAndExpand" VerticalOptions="StartAndExpand">
        	<Button Margin="0,0,0,0" Image="{Binding StepDownImage}" HorizontalOptions="Start" />
            	<customControls:ExtendedEntry x:Name="BoxQuantity" Text="{Binding StepperEntry, Mode=TwoWay}" Margin="6,-4,6,0" BackgroundColor="{StaticResource RotateEntrInputBox}" MarginBottom="2" CenterText="true" />
        </StackLayout>
```

After completing the process you will be able to use the created customization.

##  4.- Things to Keep in Mind For a Custom Renderer Implementation
#### 1. A second look
Because Custom Renderers are process intensive, solving the requirement from the Xamarin Forms level itself can save resources and processing time. See <https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/effects/introduction> This is something similar to Custom Renderers but it is more simplified and less process intensive. Always look for the alternatives before moving towards renderers.

#### 2. Reusability
When creating a Custom Renderer, it’s important to focus on reusability because of the fact that it’s process intensive. Try to merge renderers together, and keep the number of renderers as low as possibile.

#### 3. Look at available properties in the render hierarchy
Before a renderers implementation, try to take a look at the renderer hierarchy starting from the Xamarin Forms level towards the actual native control level. Go through the properties that are available at the native level in all three platforms before deciding to go use Custom Renderers.

##  5.- Reference
You can find more detailed information along with few more examples at the official Microsoft Xamarin documentation & samples web sites:

* Introduction to Custom Renderers:
<https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/custom-renderer/introduction>

* Renderer Base Classes and Native Controls:
<https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/custom-renderer/renderers>

* Introduction to Xamarin Forms Custom Renderers
<https://academy.realm.io/posts/introduction-to-xamarin-forms-custom-renderers/>

* Introduction to Effects
<https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/effects/introduction>
