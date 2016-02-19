## go-panos
[![GoDoc](https://godoc.org/github.com/scottdware/go-panos?status.svg)](https://godoc.org/github.com/scottdware/go-panos) [![Travis-CI](https://travis-ci.org/scottdware/go-panos.svg?branch=master)](https://travis-ci.org/scottdware/go-panos)
[![license](http://img.shields.io/badge/license-MIT-red.svg?style=flat)](https://raw.githubusercontent.com/scottdware/go-panos/master/LICENSE)

A Go package that interacts with Palo Alto and Panorama devices using the XML API.

Be sure to visit the [GoDoc][godoc-go-panos] page for official package documentation.

> Note: The below examples are a work-in-progress :)

### Examples

* [Connecting to a device][connecting-to-a-device]
* [Listing objects (address, service, device-groups, tags, etc.)][listing-objects]
* [Creating objects][creating-objects]
* [Deleting objects][deleting-objects]
* [Commiting configurations][commiting-configurations]

#### Connecting to a Device

Establish a connection to a Palo Alto firewall or Panorama is pretty straightforward:

```Go
pa, err := panos.NewSession("pa200-fw", "admin", "paloalto")
if err != nil {
    fmt.Println(err)
}
```

Once you are connected, some basic information about the firewall/session is established. The fields returned are as follows:

|Field|Description|
|-----|-----------|
|Host|Hostname/IP of the device.|
|Key|Encrypted key for API access.|
|URI|The base URI that all API calls will use.|
|Platform|Hardware platform of the device.|
|Model|Model number of the device.|
|Serial|Serial number of the device.|
|SoftwareVersion|Software version currently active on the device.|
|DeviceType|Type of device, i.e. "panorama" or "panos."|
|Panorama|A boolean (true/false) field that determines if the device is/isn't Panorama.| 

```Go
fmt.Printf("Host: %s\n", pa.Host)
fmt.Printf("Key: %s\n", pa.Key)
fmt.Printf("URI: %s\n", pa.URI)
fmt.Printf("Platform: %s\n", pa.Platform)
fmt.Printf("Model: %s\n", pa.Model)
fmt.Printf("Serial: %s\n", pa.Serial)
fmt.Printf("Software Version: %s\n", pa.SoftwareVersion)
fmt.Printf("Device Type: %s\n", pa.DeviceType)
fmt.Printf("Panorama Connection: %t\n", pa.Panorama)
```

#### Listing Objects

To list all address objects, use the `Addresses()` function. The fields returned are as follows: 

> Note: Some of these fields will be empty based on the type of address returned.

|Field|Description|
|-----|-----------|
|Name|Name of the object.|
|IPAddress|IP/network of the object.|
|IPRange|IP range of the object.|
|FQDN|Fully qualified domain name of the object.|
|Description|Description of the object, if available.|

```Go
addrs, _ := pa.Addresses()

for _, a := range addrs.Addresses {
    fmt.Println(a.Name)
    fmt.Println(a.IPAddress)
    fmt.Println(a.IPRange)
    fmt.Println(a.FQDN)
    fmt.Println(a.Description)
}
```

To list all address group objects, use the `AddressGroups()` function. The fields returned are as follows:

|Field|Description|
|-----|-----------|
|Name|Name of the group.|
|Type|Type of address group, i.e. "Static" or "Dynamic."|
|DynamicFilter|The filter for the dynamic address group.|
|Members|Address objects that belong to the group.|
|Description|Description of the group, if available.|

> Note: Members are in a string slice (`[]string`), so to iterate over them you can just do another loop.

```Go
addrGroups, _ := pa.AddressGroups()

for _, ag := range addrGroups.Groups {
    fmt.Println(ag.Name, ag.Type, ag.DynamicFilter, ag.Description)
    for _, m := range ag.Members {
        fmt.Println(m)
    }
}
```

To list all service objects, use the `Services()` function. The fields returned are as follows:

> Note: Some fields might be empty based on they type of service returned.

|Field|Description|
|-----|-----------|
|Name|Name of the object.|
|TCPPort|TCP port(s) of the object.|
|UDPPort|UDP port(s) of the object.|
|Description|Description of the object, if available.|

```Go
svcs, _ := pa.Services()

for _, s := range svcs.Services {
    fmt.Println(s.Name)
    fmt.Println(s.TCPPort)
    fmt.Println(s.UDPPort)
    fmt.Println(s.Description)
}
```

To list all service groups, use the `ServiceGroups()` function. The fields returned are as follows:

|Field|Description|
|-----|-----------|
|Name|Name of the object.|
|Members|Service objects that belong to the group.|
|Description|Description of the object, if available.|

> Note: Members are in a string slice (`[]string`), so to iterate over them you can just do another loop.

```Go
svcGroups, _ := pa.ServiceGroups()

for _, sg := range svcGroups.Groups {
    fmt.Println(sg.Name, sg.Description)
    for _, m := range sg.Members {
        fmt.Println(m)
    }
}
```

To list all device-groups on a Panorama device, use the `DeviceGroups()` function. The fields returned are as follows:

|Field|Description|
|-----|-----------|
|Name|Name of the object.|
|Devices|Individual devices that belong to the device-group.|

> Note: Devices (serial numbers) are in a string slice (`[]string`), so to iterate over them you can just do another loop.

```Go
devGroups, _ := pa.DeviceGroups()

for _, d := range devGroups.Groups {
    fmt.Println(d.Name)
    for _, serial := range d.Devices {
        fmt.Println(serial)
    }
}

```

To list all tags, use the `Tags()` function. The fields returned are as follows:

|Field|Description|
|-----|-----------|
|Name|Name of the tag.|
|Color|Color of the tag, i.e. "Blue" or "Cyan."|
|Comments|Description of the tag, if available.|

```Go
tags, _ := pa.Tags()

for _, t := range tags.Tags{
    fmt.Println(t.Name)
    fmt.Println(t.Color)
    fmt.Println(t.Comments)
}
```

#### Creating Objects

The `CreateAddress()` function takes 4 parameters: `name`, `address type`, `address` and an (optional) `description`. When
creating an address object on Panorama, you must specify the `device-group` to create the object in as the last/5th parameter.


> Note: The second parameter is the address type. It can be one of `ip`, `range` or `fqdn`.

```Go
pa.CreateAddress("fqdn-object", "fqdn", "sdubs.org", "My personal website")
pa.CreateAddress("Apple-subnet", "ip", "17.0.0.0/8", "")
pa.CreateAddress("IP-range", "range", "192.168.1.1-192.168.1.20", "IP address range")
```

When creating an address or service on a Panorama device, specify the desired device-group as the 
last parameter, like so:

```Go
pa.CreateAddress("panorama-IP-object", "ip", "10.1.1.5", "", "Lab-Devices")
```

The `CreateService()` function takes 4 parameters: `name`, `protocol`, `port` and an (optional) `description`. When
creating a service object on Panorama, you must specify the `device-group` to create the object in as the last/5th parameter.

> Note: The third parameter is the port number(s). Port can be a single port #, range (1-65535), or comma separated (80, 8080, 443).

```Go
pa.CreateService("proxy-ports", "tcp", "8080,80,443", "")
pa.CreateService("misc-udp", "udp", "10001-10050", "Random UDP ports")
```

Just like creating address objects on Panorama, the same applies to service objects:

```Go
pa.CreateAddress("panorama-ports", "tcp", "8000-9000", "Misc TCP ports", "Lab-Devices")
```

#### Deleting Objects

Deleting objects is just as easy as creating them. Just specify the object name, and if deleting objects from Panorama,
specify the device-group as the last/2nd parameter.

```Go
pa.DeleteAddress("fqdn-object")
pa.DeleteService("proxy-ports")
pa.DeleteAddress("some-panorama-IP", "Lab-Device-Group")
```

#### Commiting Configurations

There are two commit functions: `Commit()` and `CommitAll()`. Commit issues a normal commit on the device. When issuing `Commit()` against a Panorama device,
the configuration will only be committed to Panorama, and not an individual device-group.

Using `CommitAll()` will only work on a Panorama device, and you have to specify the device-group you want to commit the configuration to. You can
also selectively commit to certain devices within that device group by adding their serial numbers as additional parameters.

```Go
pa.Commit()

// CommitAll will commit the configuration to the given device-group, and all of it's devices.
pa.CommitAll("Lab-Device-Group")

// Commit to only 2 devices in a device group - you MUST use their serial numbers
pa.CommitAll("Lab-Device-Group", "1093822222", "1084782033")
```

[godoc-go-panos]: http://godoc.org/github.com/scottdware/go-panos
[license]: https://github.com/scottdware/go-panos/blob/master/LICENSE
[connecting-to-a-device]: https://github.com/scottdware/go-panos#connecting-to-a-device
[listing-objects]: https://github.com/scottdware/go-panos#listing-objects
[creating-objects]: https://github.com/scottdware/go-panos#creating-objects
[deleting-objects]: https://github.com/scottdware/go-panos#deleting-objects
[commiting-configurations]: https://github.com/scottdware/go-panos#commiting-configurations