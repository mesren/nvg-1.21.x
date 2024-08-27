## NanoVG for 1.21x

Tutorial inspired by (this)[https://nanovg.aetherium.club/gradle-mcp.html]. (This wont work on its own)

# Getting and patching LWJGL3

WaterwaveMC has a patched LWJGL, so we're going to use that.

Add this to your gradle.properties:
```groovy
# LWJGL Properties
lwjgl_version = 3.3.2
lwjgl_patch_version = 5
```

Add this to your build.gradle:
```
repositories {
        maven { url "https://jitpack.io" }
        mavenCentral()
}

dependencies {
    implementation platform("org.lwjgl:lwjgl-bom:3.3.2")

    runtimeOnly "org.lwjgl:lwjgl::$lwjglNatives"
    runtimeOnly "org.lwjgl:lwjgl-nanovg::$lwjglNatives"
    runtimeOnly "org.lwjgl:lwjgl-stb::$lwjglNatives"

    implementation "io.waterwave.Legacy-LWJGL3:lwjgl:${project.lwjgl_version}-${project.lwjgl_patch_version}"
    implementation "io.waterwave.Legacy-LWJGL3:lwjgl-stb:${project.lwjgl_version}-${project.lwjgl_patch_version}"
    implementation "io.waterwave.Legacy-LWJGL3:lwjgl-nanovg:${project.lwjgl_version}-${project.lwjgl_patch_version}"
}
```
Reload your project and see if there are any errors.
# Patching Nano VG
We used a pre-patched LWJGL, However NanoVG isn't patched so we will through a Mixin.

NanoVG looks for a GL class, but it's not present in LWJGL 2, so we have to use our own function provider.

(Function Provider)[https://gist.github.com/refactorinqq/31a10444bb6c49eb5a370b2353f664cb]
(Mixin)[https://gist.github.com/refactorinqq/b65f3ac1aba2cf3a15323aa60f47995f]

Congrats! You have successfully aqired LWJGL and NanoVG.
# UI Example
```java
import static org.lwjgl.nanovg.NanoVG.*;
import static org.lwjgl.nanovg.NanoVGGL2.*;
import static org.lwjgl.opengl.GL11.*;

private long ctx = -1L;

/**
* ONLY call this ONCE!!!
*/

// The author of this class is refactoring, NOT ME!

public void init() {
    ctx = nvgCreate(NVG_ANTIALIAS | NVG_STENCIL_STROKES); // NanoVGGL2 must be used.
        
    if(ctx == -1) {
         throw new IllegalStateException("Context not initialized");
    }
}

public void render() {
    glPushAttrib(GL_ALL_ATTRIB_BITS);
    GlStateManager.disableAlphaTest();
    nvgBeginFrame(ctx, Display.getWidth(), Display.getHeight(), 1F);
    try(NVGColor col = NVGColor.calloc()) {
        nvgRGBf(255, 255, 255, col); // White
        nvgBeginPath(ctx);
        nvgRoundedRect(ctx, 10f, 10f, 100f, 100f, 10f);
        nvgFillColor(ctx, col);
        nvgFill(ctx);
    }
    nvgEndFrame(ctx);
    GlStateManager.enableAlphaTest();
    glPopAttrib();
}
```
