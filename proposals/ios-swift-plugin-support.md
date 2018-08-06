# Swift Plugin support
- Status: Proposed

The purpose of this proposal is to support ios plugin written by swift.
I propose introducing two functions. One is editing bridging-header according to plugin.xml settings. The other is select
application swift version according to config.xml.
Note that we can specify swift version for the application. Plugins with different swift version can not imported simultaneously. This is contrary with cocoapod libraries in which swift version can be specified for each.

## Bridiging-Header

Introduce the `bridging-header` attribute for the `header-file` tag in plugin.xml

ex.
```
<header-file src="src/ios/HOge-Briding-Header.h" bridging-header="true" />
```
This attribute accepts boolean values. By default, when undefined, it is set to false.

When the briding-header attribute is set to `true`, the `Briding-Header.h` file, located in cordova-ios template is updated as

```
#import <Cordova/CDV.h>
#import "Plugins/cordova-plugin-xxxx/Hoge-Bridging-Header.h"
```

This new feature, i.e. updating `Briding-Header.h` file, is working at 

`after plugin add`
`after plugin rm`

## Select application swift version

Introducing the SwiftVersion for preference tag in config.xml.

ex.
```
<preference name="SwiftVersion" value="4.1" />
```

This is working to specify swift version in the application at

`after prepare`

by using xcode module.