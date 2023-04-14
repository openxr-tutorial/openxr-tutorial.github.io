# Creating an XrInstance

To use OpenXR, we must first initialize it by creating an _OpenXR instance_.
The instance is the connection between your application and the OpenXR runtime, and when
creating it we must pass the runtime details about our application.

First, we'll fill in an `XrApplicationInfo` struct with information about our application. 
This data will be used by the driver to present useful information, such as in the runtime's overlay.

```c
XrApplicationInfo application_info = {
    .apiVersion = XR_CURRENT_API_VERSION,
    .applicationName = "OpenXR Tutorial",
    .applicationVersion = 1,
    .engineName = "OpenXR Tutorial Engine",
    .engineVersion = 1,
};
```

We're using designated initialization, which allows us to conveniently intitialise the
`applicationName` and `engineName` fields, as well as automatically _zero initializing_ any fields
we did not specify explictly.

Most information in OpenXR is passed through structs rather than function parameters. Creating
an instance requires us to fill out one more struct: `XrInstanceCreateInfo`. This tells the driver
which API layers and Extensions that we wish to use. For now, we'll be leaving those fields blank
and just passing in the application info.

```c
XrInstanceCreateInfo instance_info = {
    .type = XR_TYPE_INSTANCE_CREATE_INFO,
    .applicationInfo = application_info,
};
```

This struct has a field `type` which is common among all structs passed directly to OpenXR procedures.
Now that we have our structs filled out, we can call `xrCreateInstance` to create the instance. Once
we've called that, we'll check that the result is equal to `XR_SUCCESS` before continuing, and exit
if it is not.

```c
XrInstance instance;
XrResult result = xrCreateInstance(&instance_info, &instance);
if (result != XR_SUCCESS) {
    printf("Failed to Create Instance\n");
    return -1;
}
```

If your program exited with an error code above, you may have also seen something printed in your
terminal such as:

```
Error [GENERAL | xrCreateInstance | OpenXR-Loader] : LoaderInstance::CreateInstance chained CreateInstance call failed
Error [GENERAL | xrCreateInstance | OpenXR-Loader] : xrCreateInstance failed
```

Even without a debug messenger enabled, the OpenXR Loader will print some minimal information such
as this. On desktop, the likeliest explanation for this error is simply that no VR HMD is plugged
into your computer. Runtimes such as SteamVR will refuse to even initialise OpenXR if no HMD is
detected, so make sure yours is plugged in.