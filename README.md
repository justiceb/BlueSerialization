![Logo](https://github.com/justiceb/BlueSerialization/blob/main/Images/24.png?raw=true)
  <h3 align="center">Blue Serialization</h3>
  <p align="center">
    Serialize and deserialize LabVIEW classes using either JSONtext or TOML
    <br />
    <a href="https://github.com/justiceb/BlueSerialization"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/justiceb/BlueSerialization">View Demo</a>
    ·
    <a href="https://github.com/justiceb/BlueSerialization/issues">Report Bug</a>
    ·
    <a href="https://github.com/justiceb/BlueSerialization/issues">Request Feature</a>
  </p>
</p>



<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary><h2 style="display: inline-block">Table of Contents</h2></summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgements">Acknowledgements</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

[![Product Name Screen Shot][product-screenshot]](https://example.com)

Here's a blank template to get started:
**To avoid retyping too much info. Do a search and replace with your text editor for the following:**
`justiceb`, `BlueSerialization`, `twitter_handle`, `bjustice@blueorigin.com`, `Blue Serialization`, `Serialize and deserialize LabVIEW classes using either JSONtext or TOML`


### Built With

* []()
* []()
* []()



<!-- GETTING STARTED -->
## Getting Started

This repository only houses the core for the framework.  You first need to decide which type of serializer that you want to use.  At the moment, the framework has the following supported serializers:

| Text Type  |  VIP Name | VIP dependency |
| ------------- | ------------- | ------------- |
| JSON  | [BlueJSONTextSerializer](https://github.com/justiceb/BlueJSONTextSerializer)  | JSONtext |
| TOML  | [BlueTOMLSerializer](https://github.com/justiceb/BlueTOMLSerializer)  | LV-TOML |

Installing one of these packages will pull down all of the dependencies that you need.

## User Guide
### How to make a serializable class

 1. Open/create a LabVIEW class. Inherit your class from BlueSerializable.lvclass. (You can drop something from the BlueSerializable palette down onto your code to bring the class into memory such that it can be inherited from.)
 2. Learn the project provider menu items, listed below:

![enter image description here](https://github.com/justiceb/BlueSerialization/blob/main/Images/1.png?raw=true)

### "Create Serializable Methods"
This script will behave differently depending on whether or not it's the first time that you have run this script against your class.

-  The first time that this script runs against your class, the following work will be performed:
	- "SerializableData.clt" will be created and added to your class's private data
		- This will be named "SerializableData" within your private data.
		- Other items in your private data with this name will be forcibly rename.
		- Please, never change this filename
	- GetSerializableData.vi will be created and added
	- SetSerializableData.vi will be created and added
	- Increment Class version major history by 1
	- Initialize your BlueSerializableVersion within the class mutation history to 1
- The second time that this script runs against your class, the following work will be performed:
	- delete, and rescript creation of the Get/SetSerializableData.vi. (We did this in case we ever needed to update these methods and fix all the VIs out in the world.)

When this script finishes, you should see the following new files:

![enter image description here](https://github.com/justiceb/BlueSerialization/blob/main/Images/2.png?raw=true)

A few notes:

-   The Get/SetSerializableData VIs are dynamic overrides. These provide access to the SerializableData in the private data. You should theoretically never have to modify these VIs. In fast, please DON'T modify these VIs unless you know what you are doing and have a good reason.
-   The SerializableData.ctl provides access to the serializables for your class. This allows you to easily separate serializables from protected private data that you don't wan to be serialized/deserialized.
-   Do NOT change the filename for SerializableData.ctl

### "Create Mutation" (First call)
This script will allow for you to change your serializable data in a way that would normally create a backwards incompatible jump with respect to old serialized text.

When the script finishes, you should see the following new files in your project:

![enter image description here](https://github.com/justiceb/BlueSerialization/blob/main/Images/3.png?raw=true)

- GetVersionSerializableData.vi
	- You theoretically never have to touch this file
- HandleVersionMutation.vi
	- You theoretically never have to touch this file
- SerializableData_v1.ctl
	- This is essentially a tagged version of SerializableData.ctl at the time of running the Create Mutation script.
	- Do NOT change this filename.
	- Do NOT change the contents of this ctl.
	- Note that ALL typedefs in this file will have been disconnected. This prevents mutations in the current-generation SerializableData from affecting the old tagged version of the data
- Mutate_v1_to_v2.vi
	- This is where you are required to add/write custom code. You will need to write code that defines the conversion between the old serialzable data and the current serializable data
	- Note that you may need to revisit this code in the future if you update SerializableData in a way that breaks the code conversion. This is expected.

### "Create Mutation" (non-First call)
When the script finishes, you should see the following new files in your project:

![enter image description here](https://github.com/justiceb/BlueSerialization/blob/main/Images/4.png?raw=true)

- SerializableData_v2.ctl
	- This is a new data structure tag... just like previously discussed
- Mutate_v2_to_v2.vi
	- This is a new mutation conversion VI... just like previously discussed

A few notes:
-   In this example, the script will actually modify the Mutate_v1_to_v2.vi. We replace the SerializableData.ctl with SerializableData_v1.vi.
-   You can mutate a class as many times as you'd like.
-   Serialized text being converted from v1 to v3 will pass through both mutation VIs. (mutate_v1_to_v2.vi and mutate_v2_to_v3.vi). This is fine. However, it is technically allowable for users to directly modify the "HandleVersionMutation.vi" in a way that will allow you to write code that will convert v1 data directly to v3 data. I don't recommend doing this unless you know what you are doing, and have a good reason.

### "View Mutation History"
This is a visualization tool and will not modify your class data or version in any manner.
This menu item appears for all classes... This is not strictly a BlueSerializable.lvclass tool.

![enter image description here](https://github.com/justiceb/BlueSerialization/blob/main/Images/5.png?raw=true)

## API
simple example:
![enter image description here](https://github.com/justiceb/BlueSerialization/blob/main/Images/6.png?raw=true)

Palette:

![enter image description here](https://github.com/justiceb/BlueSerialization/blob/main/Images/7.png?raw=true)

### API - Serialize
![enter image description here](https://github.com/justiceb/BlueSerialization/blob/main/Images/8.png?raw=true)

-   Input. This can be any LabVIEW data structure. This can be as complex as you'd like... nested clusters, classes, arrays, etc.
    -   NOTE: Don't ask me "does BlueSerialization support Map Collections, or XYZ datatypes? The answer to this question lies in whether or not the selected serializer supports this data type. For example, JSONtext does not support map collections. The BlueSerialization provides a framework for decomposing/composing LabVIEW class instances and then leveraging the selected serializer for handling all other data types.
-   Output. This is serialized text. If you used a JSONtext serializer, then this will be JSON text.

| Category  |  Name | Default Value | Description |
| ------------- | ------------- | ------------- | ------------- |
| class slicing | slice objects per SetSerializeSliceClass.vi?  | False | If True, then the wired object will be sliced to the class layer specified in the "SetSerializeSliceClass override. (No-op parent will not perform any slicing.)|
| non-Blue | serialize non-BlueSerializable objects as hex?  | False | If True, then non-BlueSerializable objects will be flattened as hex data using the LabVIEW "flatten to string" node. If false, an error will be returned when a non-BlueSerializable object is discovered |

### API - Deserialize
![enter image description here](https://github.com/justiceb/BlueSerialization/blob/main/Images/9.png?raw=true)

-   Serialized Object String. This is your serialized text (JSON, TOML, etc.)
-   Anything. This is your reference data structure that you wish to adapt the serialized text into.
-   Variant. Use the "Variant to data" node to convert this variant into a LabVIEW datatype. Generally, you will use the wire-type of the "Anything" input.

| Category  |  Name | Default Value | Description |
| ------------- | ------------- | ------------- | ------------- |
| expansion | expand objects on type conflict?  | False | If True, then the wired object is allowed to be more-specific than the serialized text. The returned object will be the more specific object. If false, then an error will be returned in this situation. |
| class slicing | slice flattened object data on error?  | False | If True, then the framework will iterate upwards through a class hierarchy until it finds a load-able class instance. This means that the returned data may be less-specific than the serialized text. |
| class slicing | sliceable class names  | empty str array |  If a class isn't found in memory, then the framework will be allowed to slice class names in this list until finding a class in the hierarchy that is load-able. |
| non-Blue | deserialize non-BlueSerializable objects from hex?  | False |  If True, then the framework will unflatten non-BlueSerializable objects using the "unflatten from text" node. If False, then non-BlueSerializable objects found in the reference data will yield and error. |
| Mutation | error on inserted parent levels?  | False |  If False, then the framework allows for class mutation history where a parent level was added. This means that your output data will contain default-data for this added class layer. If True, then error |
| Mutation |  error on  obsolete  parent levels? | False |  If False, then the framework allows for class mutation history where a parent level was removed. This means that data for dropped class layers in your serialized text will also be unused. If True, then error. |
| Data preservation | Preserve wired non-serialized data? | Don't Preserve | "Don't Preserve": non-serialized data will not be preserved. <br> "Preserve (error on less-specific wired types)":  non-serialized data will be preserved. However, if a wired object is found to be less specific than the serialized object, then you will receive an error. <br> "Preserve (except on less-specific wired types)":  non-serialized data will be preserved except where a wired object is found to be less specific than the serialized object. (It's not possible to preserve data in this situation without class composition.)|

### API - Set Serialize Slice Class
![enter image description here](https://github.com/justiceb/BlueSerialization/blob/main/Images/10.png?raw=true)

-   This is used in conjunction with the serializer "Seriualize.Allow class slicing" option. (See above.)
-   If class slicing is enabled on serialize, then the framework will slice to this class name






1. Clone the repo
   ```sh
   git clone https://github.com/justiceb/BlueSerialization.git
   ```
2. Install NPM packages
   ```sh
   npm install
   ```



<!-- USAGE EXAMPLES -->
## Usage

Use this space to show useful examples of how a project can be used. Additional screenshots, code examples and demos work well in this space. You may also link to more resources.

_For more examples, please refer to the [Documentation](https://example.com)_



<!-- ROADMAP -->
## Roadmap

See the [open issues](https://github.com/justiceb/BlueSerialization/issues) for a list of proposed features (and known issues).



<!-- CONTRIBUTING -->
## Contributing

Contributions are what make the open source community such an amazing place to be learn, inspire, and create. Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request



<!-- LICENSE -->
## License

Distributed under the MIT License. See `LICENSE` for more information.



<!-- CONTACT -->
## Contact

Your Name - [@twitter_handle](https://twitter.com/twitter_handle) - bjustice@blueorigin.com

Project Link: [https://github.com/justiceb/BlueSerialization](https://github.com/justiceb/BlueSerialization)



<!-- ACKNOWLEDGEMENTS -->
## Acknowledgements

* []()
* []()
* []()





<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/justiceb/repo.svg?style=for-the-badge
[contributors-url]: https://github.com/justiceb/repo/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/justiceb/repo.svg?style=for-the-badge
[forks-url]: https://github.com/justiceb/repo/network/members
[stars-shield]: https://img.shields.io/github/stars/justiceb/repo.svg?style=for-the-badge
[stars-url]: https://github.com/justiceb/repo/stargazers
[issues-shield]: https://img.shields.io/github/issues/justiceb/repo.svg?style=for-the-badge
[issues-url]: https://github.com/justiceb/repo/issues
[license-shield]: https://img.shields.io/github/license/justiceb/repo.svg?style=for-the-badge
[license-url]: https://github.com/justiceb/repo/blob/master/LICENSE.txt
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://linkedin.com/in/justiceb
