# Creating a Debug Messenger

When writing OpenXR application code, it's inevitable that we'll make the occasional mistake.
Since OpenXR is rigorously specified, correct usage of OpenXR procedures can, to an extent,
be programmatically verified. 

Doing this by default would, however, impose too large a performance penalty on applications.
So, during development, we can optionally enable debug utils to help us write correct code.
This is done with the `XR_EXT_debug_utils` extension. To enable the extension, we must modify
our call to `xrCreateInstance` to include the name of the extension. Since this is, for now,
our only extension, we'll put the extension count behind a preprocessor check, to only enable
it during debug builds.

```c
#ifndef NDEBUG
const char *enabled_extensions[] = {XR_EXT_DEBUG_UTILS_EXTENSION_NAME};
uint32_t extension_count = 1;
#else
const char **enabled_extensions = NULL;
uint32_t extension_count = 0;
#endif

XrInstanceCreateInfo instance_info = {
    .type = XR_TYPE_INSTANCE_CREATE_INFO,
    .applicationInfo = application_info,
    .enabledExtensionCount = extension_count,
    .enabledExtensionNames = enabled_extensions,
};
```

With the extension enabled, we can now create a Debug Utils Messenger, which will be referenced by
a `XrDebugUtilsMessengerEXT` handle. To create one, we must provide the OpenXR runtime with a callback,
which will be invoked whenever a debug message is produced. Inside this callback, we can choose to
do whatever we wish. Here, we will print the information with `printf`, another option would be writing
messages to a logfile.

The function signature of the callback is declared in the OpenXR headers as `PFN_xrDebugUtilsMessengerCallbackEXT`.
We can copy that signature to define our function. In C++, `extern "C"` should be prepended to the declaration
to ensure it has C linkage.

```c
XrBool32 openxr_debug_callback(
        XrDebugUtilsMessageSeverityFlagsEXT              messageSeverity,
        XrDebugUtilsMessageTypeFlagsEXT                  messageTypes,
        const XrDebugUtilsMessengerCallbackDataEXT*      callbackData,
        void* userData) {
    printf("XR DEBUG MESSAGE: %s\n", callbackData->message;
    return XR_FALSE;
}
```

The return value of this callback is an `XrBool32` indicating whether the calling layer should abort the call of
the function that produced the debug callback. This can be useful to abort execution if, for example, `messageSeverity`
contains the `XR_DEBUG_UTILS_MESSAGE_SEVERITY_ERROR_BIT_EXT` bit. Here, we simply always continue execution.
