Android Camera设置预览方向与拍摄方向
=================================

Camera方向与角度：
- 屏幕坐标方向
- 屏幕方向与角度
- 摄像头传感器方向与角度
- 预览方向与旋转角度
- 拍摄方向与旋转角度

屏幕坐标方向
===========

在Android系统中，以屏幕左上角为坐标系统的原点(0,0)坐标，该坐标系是固定不变的，不会因为设备方向的变化而改变。
```
┌────────────────────┐            
│ ┼────────────────▶ │
│ │(0,0)           x │
│ │                  │
│ │                  │
│ │                  │
│ │                  │
│ │                  │
│ │                  │
│ │                  │
│ ▼ y                │
└────────────────────┘
屏幕坐标方向
```

屏幕方向与角度
=======

通常屏幕方向也就是手机的自然方向。在Android设备中，手机的自然方向为纵向(`portrait`)0度，Pad的方向为横向（`landscape`）90度。
默认情况，自然状态下手机的屏幕方向为0度。
```
┌───────────────────┐
│                   │
│   ◤            ◐  │
│        ╱ ╲        │
│         │         │
│         │         │
│         │         │
│         │         │
│                   │
│                   │
└───────────────────┘
屏幕方向（竖屏0度）
```

获取屏幕方向角度
``` java
    // 获取屏幕方向角度
    int getScreenOrientationDegrees(@NonNull Context context) {
        int rotation;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
            rotation = Objects.requireNonNull(context.getDisplay()).getRotation();
        } else {
            WindowManager windowManager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
            rotation = windowManager.getDefaultDisplay().getRotation();
        }
        int degrees = 0;
        switch (rotation) {
            case Surface.ROTATION_0:
                degrees = 0;
                break;
            case Surface.ROTATION_90:
                degrees = 90;
                break;
            case Surface.ROTATION_180:
                degrees = 180;
                break;
            case Surface.ROTATION_270:
                degrees = 270;
                break;
            default:
                break;
        }
        return degrees;
    }
```

摄像头传感器方向与角度
==============


> suppose a device has a naturally tall screen. The back-facing camera sensor is mounted in landscape. You are looking at the screen. If the top side of the camera sensor is aligned with the right edge of the screen in natural orientation, the value should be 90. If the top side of a front-facing camera sensor is aligned with the right of the screen, the value should be 270.``

> 假设一个设备有一个纵向屏幕。背面摄像头传感器安装在横向上。您正在查看屏幕。如果相机传感器的顶部与自然方向下屏幕的右边缘对齐，则该值应为90。如果前置摄像头传感器的顶部与屏幕右侧对齐，则该值应为270。


如上属述，Android设备的后置摄像头方向（`CameraInfo.orientation`）为90度，前置摄像头方向为270度，且这个值恒定不变。此时屏幕与传感器的角度关系如下图：
```
┌───────────────────┐
│                   │
│   ◤            ◐  │
│        ╱ ╲        │
│         │         │
│         │         │
│         │         │
│         │         │
│                   │
│                   │
└───────────────────┘
手机自然方向0度

┌─────────────────────────────────┐
│     ◒                           │
│          /                      │
│            ────────────────     │
│          \                      │
│     ◣                           │          
└─────────────────────────────────┘
后置摄像头传感器方向（90度）

┌─────────────────────────────────┐
│                          ◥      │
│                    \            │
│       ─────────────             │
│                    /            │
│                          ◓      │          
└─────────────────────────────────┘
前置摄像头传感器方向（270度）
```

预览方向与旋转角度
================

预览图像时，通过旋转预览方向，使预览图像方向与屏幕方向一致，这个方向称之为预览方向。

后置摄像头
---------
对于竖屏来说，相对于屏幕方向传感器方向逆时针旋转了90度，因此需要将预览方向顺时针90度，才能与屏幕方向一致。
对于横屏来说，传感器方向与屏幕方向一致，预览方向不需要旋转角度。

前置摄像头
---------
前置摄像头的传感器角度为270度，相对于屏幕竖屏方向顺时针旋转了90度，因此预览方向需要顺时针旋转270度才能与屏幕方向一致。但是``前置摄像头预览方向在旋转角度之前，会先进行镜像处理（顺时针旋转180度）``，因此只需顺时针旋转90度，既可以与屏幕方向一致。

设置预览方向旋转角度
``` java
    /**
     * 计算预览方向顺时针旋转角度
     * @param cameraInfo               相机信息
     * @param screenOrientationDegrees 屏幕方向角度
     * @return 传感器图像顺时针旋转角度
     */
    int calcDisplayOrientation(Camera.CameraInfo cameraInfo, int screenOrientationDegrees) {
        if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_FRONT) {
            return (360 - (cameraInfo.orientation + screenOrientationDegrees) % 360) % 360;
        } else {  // back-facing
            return (cameraInfo.orientation - screenOrientationDegrees + 360) % 360;
        }
    }
    
    ...
    设置预览方向顺时针旋转角度
    int displayOrientation = calcDisplayOrientation(cameraInfo, screenOrientationDegrees);
    camera.setDisplayOrientation(displayOrientation);
```

拍摄方向与旋转角度
===================

> Sets the clockwise rotation angle in degrees relative to the orientation of the camera. This affects the pictures returned from JPEG Camera.PictureCallback. The camera driver may set orientation in the EXIF header without rotating the picture. Or the driver may rotate the picture and the EXIF thumbnail. If the Jpeg picture is rotated, the orientation in the EXIF header will be missing or 1 (row #0 is top and column #0 is left side).
> 
> If applications want to rotate the picture to match the orientation of what users see, apps should use android.view.OrientationEventListener and Camera.CameraInfo. The value from OrientationEventListener is relative to the natural orientation of the device. CameraInfo.orientation is the angle between camera orientation and natural device orientation. The sum of the two is the rotation angle for back-facing camera. The difference of the two is the rotation angle for front-facing camera. Note that the JPEG pictures of front-facing cameras are not mirrored as in preview display.
> 
> For example, suppose the natural orientation of the device is portrait. The device is rotated 270 degrees clockwise, so the device orientation is 270. Suppose a back-facing camera sensor is mounted in landscape and the top side of the camera sensor is aligned with the right edge of the display in natural orientation. So the camera orientation is 90. The rotation should be set to 0 (270 + 90).

> 设置相机的顺时针旋转角度（以度为单位），相对于相机的方向。这会影响从JPEG Camera.PictureCallback返回的图片。相机驱动程序可能在EXIF头中设置方向而不旋转图片。或者驱动程序可能旋转图片和EXIF缩略图。如果Jpeg图片被旋转，EXIF头中的方向将丢失或为1（第0行是顶部，第0列是左侧）。
> 
> 如果应用程序想要旋转图片以匹配用户所看到的方向，应该使用android.view.OrientationEventListener和Camera.CameraInfo。OrientationEventListener的值是相对于设备的自然方向而言的。CameraInfo.orientation是摄像头方向与设备自然方向之间的角度。两者之和是后置摄像头的旋转角度。两者之差是前置摄像头的旋转角度。请注意，前置摄像头拍摄出来的JPEG照片在预览显示中不会镜像。
> 
> 例如，假设设备的自然方向为纵向（portrait）。设备顺时针旋转270度，因此设备方向为270。假设一个后置摄像头传感器安装在横向（landscape）上，并且相机传感器的顶部与自然方向下显示屏右边缘对齐。因此相机方向为90。旋转角度应设置为0（270 + 90）。

设置预览方向不会改变拍摄照片或者视频的方向。
对于后置相机，拍摄方向与预览方向类似，只需要顺时针旋转90度。
对于前置相机，拍摄方向与预览方向类似，但是拍摄方向在旋转角度时不会执行镜像操作，所以需要顺时针旋转270度。
由于设备可以倒转方向，因此计算角度时还需要加上`翻转角度（landscapeFlip）`。

设置拍摄方向旋转角度
``` java
    /**
     * 计算图像输出顺时针旋转角度
     * @param cameraInfo               相机信息
     * @param screenOrientationDegrees 屏幕方向角度
     * @return 顺时针旋转角度
     */
    int calcCameraRotation(Camera.CameraInfo cameraInfo, int screenOrientationDegrees) {
        if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_FRONT) {
            return (cameraInfo.orientation + screenOrientationDegrees) % 360;
        } else {  // back-facing
            final int landscapeFlip = isLandscape(screenOrientationDegrees) ? 180 : 0;
            return (cameraInfo.orientation + screenOrientationDegrees + landscapeFlip) % 360;
        }
    }
    
    ...
    // 设置相机的顺时针旋转角度（以度为单位），相对于相机的方向。这会影响从JPEG Camera.PictureCallback返回的图片。
    Camera.Parameters parameters = camera.getParameters();
    int rotation = calcCameraRotation(cameraInfo, screenOrientationDegrees);
    parameters.setRotation(rotation);
```

参考
====
https://www.jianshu.com/p/b5d8e0e13cd9

https://yeungeek.github.io/2020/01/24/AndroidCamera-Orientation/
