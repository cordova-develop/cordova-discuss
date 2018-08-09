# Cocoapod Improvement
- Status: Proposed

## Overview

The purpose of this proposal is to improve Cocoapod support.
This option would basically deprecate the framework usage of type=”podspec”.
The benefits would allow better readability. Add default additional configuration settings as arguments to the to the pods element. For example <pods use-frameworks=true> but the use_framework! option is not required anymore since 1.5.0.

ex.
```
<podspec>
  <config>
    <source url="https://github.com/brightcove/BrightcoveSpecs.git" />
    <source url=”https://github.com/CocoaPods/Specs.git”/>
  </config>

  <pods>
    <pod name=”PromiseKit” />
    <pod name=”Foobar1” spec=”~> 2.0.0” />
    <pod name=”Foobar2” git=”git@github.com:hoge/foobar1.git” />
    <pod name=”Foobar3” git=”git@github.com:hoge/foobar2.git” branch=”next” />
    <pod name=”Foobar4” swift-version=”4.1” />
    <pod name=”Foobar5” swift-version=”3.0” />
  <pods>
</podspec>
```

## The newly introduced tags

- `<podspec>` tag  
  available in global scope.  
  having no attributes.  
  having body consisting of `<config>` tag, `<pods>` tag.  

- `<config>` tag  
  available in `podspec` scope.  
  having no attributes.  
  having body consisting of `<source>` tag.   

- `<pods>` tag  
  available in `podspec` scope.  
  having “use-frameworks” attribute.    
  having “inhibit_all_warnings” attribute.  
  having body consisting of `<pod>` tag.  


- `<source>` tag  
   available in `config` scope.  
   having  url attribute.  
   having no body.  

- `<pod>` tag  
   available in `pods` scope.  
   having `name`, `spec`, `tag`, `branch`, `commit`, `configurations`, `git`, `http`, podspec, path, swift-version,   options  attributes  


ex.1
```
<pod name=”GoogleAnalytics” spec=”~> 3.1” />
```
in plugin.xml
becomes
```
pod 'GoogleAnalytics', '~> 3.1'
```
in Podfile.


ex.2
```
<pod name=”Alamofire” path=”~/Documents/Alamofire” />
```
in plugin.xml
becomes
```
 pod ‘Alamofire’, :path => ‘~/Documents/Alamofire’
```
in Podfile.


The attribute ‘options’ is a key-value list like

```
<pod name=”Alamofire” options=”:git => 'https://github.com/Alamofire/Alamofire.git', :tag => '3.1.1'” />
```

This is because there are so many features in CocoaPod. It is difficult to keep track of all.  Most of them have not been used much.
For example, CocoaPod supports svn (:svn) and its head (:head), mercurial (:hg) and  bazaar (:bzr).



## Extending pods.json

The current pods.json file records only cocoapods libraries' infomation, such as

```
{
   "SwiftMessages": {
       "name": "SwiftMessages",
       "type": "podspec",
       "spec": "~> 4.1",
       "count": 1
   }
}
```

Extending this pods.json file to manage three sections. The one section is for configs, the another section is for sources and the last one is for libraries.

```
{
    "configs": { 
      "use-frameworks: true" : {
        "config": "use-frameworks: true",
        "count": 1
      }, 
      "inhibit_all_warnings!": {
        "config": "inhibit_all_warnings!",
        "count": 1
      } 
    },
    "sources": {
"https://github.com/CocoaPods/Specs.git": {
	  "source" : "https://github.com/CocoaPods/Specs.git",
	  "count" : 1
	},
      …
    },
    "libraries": {
      "SwiftMessages": {
        "name": "SwiftMessages",
        "type": "podspec",
        "spec": "~> 4.1",
	      "swift-version": “4.1”,
        "count": 1
      }
    }
}
```

The count parameter indicates the number of plugins having Podfile settings.

Podfile is modified according to this pods.json file. All the elements with count > 0 is added in Podfile.  

When the developer removes a plugin, the corresponding count parameter is decreased. If the count becomes zero, the element is removed.

If the same key with different element is about to be added, only count parameter is increased for previous element with same key and show warnings.

For example pluginA has

```
<podspec>
  <pods>
    <pod name=”Foobar1” spec=”~> 2.0.0” />
  <pods>
</podspec>
```

and pluginB has

```
<podspec>
  <pods>
    <pod name=”Foobar1” spec=”~> 3.0.0” />
  <pods>
</podspec>
```

If the developer adds pluginA and pluginB in this order, the resulting pods.json becomes

```
{
    "configs": { 
    },
    "sources": {
	  },
    "libraries": {
      "Foobar1": {
        "name": "Foobar1",
        "type": "podspec",
        "spec": "~> 2.0.0",
        "count": 2
      }
    }
}
```


## Compatibility

The framework tag with type=”podspec” is still available for the compatibility.

```
   <framework src="SwiftMessages" type="podspec" spec="~> 4.1" />
```

Combining new and old formula in plugin.xml, then update Podfile.
