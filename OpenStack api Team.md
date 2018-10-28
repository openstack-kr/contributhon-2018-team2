# OpenStack Document - api Team

- 강상규 
- 조정연
- 이재연

<br/>

## OpenStack의 개념과 구성요소

### Openstack이란?

OpenStack은 클라우드 환경에서 컴퓨팅 자원과 스토리지 인프라를 셋업하고 구동하기 위해 사용하는 Opensource Software Project의 집합이다.

IaaS 형태의 클라우드 컴퓨팅 오픈소스 프로젝트로 컴퓨팅, 스토리지, 네트워킹 자원을 관리하는 여러개의 하위 프로젝트들로 이루어져있다.

<br/>

### OpenStack 구성요소

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/openstack%20%E1%84%89%E1%85%A5%E1%86%AF%E1%84%80%E1%85%A8%E1%84%83%E1%85%A9.png"/>

#### - Nova (Compute)

서버 가상화를 지원하는 것으로 여러 가상화 SoftWare를 제어하고 Virtual Machine을 관리한다.

#### - Swift (Object Storage)

Rest API를 이용하여 필요한 OS 이미지를 관리한다. 파일 단위로 데이터를 저장하며, HTTP/HTTPS 프로토콜을 사용하는 서버이므로, 외부에서 URL을 이용하여 접근할 수 있다.

#### - Glance (Image)

Nova에서 VM을 만들 때, 필요한 OS 이미지를 관리한다.

#### - Keystone

서비스를 허용된 사용자만 사용할 수 있도록 인증하는 과정을 통합적으로 관리한다.

논리 아키텍처에서는 Keystone은 다음으로 구성된다.

1. Token Backend : 사용자별 Token 관리
2. Catalog Backend : OpenStack의 모든 서비스의 Endpoint URL을 관리
3. Policy Backend : Tenant, 사용자 계정 및 룰을 관리
4. Identity Backend : 사용자 인증 관리

#### - Neutron

네트워크 서비스를 지원하며, SDN(Software Define Network)의 첫 번째 사례이다.

#### - Telemetry

OpenStack에서 제공되는 자원의 사용량을 관리하며, 최근에는 모니터링 용도로 확장되고 있다.

<br/>

## API 연습하기

API URL은 ``openstack horizon`` > ``관리자`` > ``시스템`` > ``시스템 정보`` 에 가면 각 Component별 url이 정의되어있다. 각 component에 맞는 url을 참고하면서 api를 테스트하면 된다.

현재 api는 **pike version 3** 을 사용하여 테스트하였다.

## 1. Keystone

### - Token 발급하기

**Request**

```
URL :  {ip:port}/identity/v3/auth/tokens
Method : Post
body :
{ "auth": {
    "identity": {
      "methods": ["password"],
      "password": {
        "user": {
          "name": "{user id}",
          "domain": { "id": "{domain id}" },
          "password": "{password}"
        }
      }
    },

   "scope": {
     "domain": {
       "id": "{domain id}"
     }
   }
  }
}  
```

**Response**

```
Header :
X-Subject-Token gAAAAABbyvJjfU7efP0Un_Npeg0Pbkp94z_7QFeOWv4wOwrrFm4YMTmKR3jQdVg5sgTjVzjD0sL9Qmu8Lw6np0UELfTGZZeUMccw1llea52UOSQj0Af151MAXGaLseQv8k3tDepewPRjpw60IkfatuEesxcYnzT6yg
```

**X-Subject-Token** 의 값이 Token이므로 이 값을 저장해 놓아야한다.

```
Body :
{
    "token": {
        "domain": {
            "id": "default",
            "name": "Default"
        },
        "methods": [
            "password"
        ],
        "roles": [
            {
                "id": "18ecfac73b224145bbf4766be03e75a0",
                "name": "admin"
            }
        ],
        "expires_at": "2018-10-20T10:16:19.000000Z",
        "catalog": [
            {
                "endpoints": [
                    {
                        "url": "http://10.0.2.15/placement",
                        "interface": "public",
                        "region": "RegionOne",
                        "region_id": "RegionOne",
                        "id": "e758f953821c43fe9c61aef60209fd05"
                    }
                ],
                "type": "placement",
                "id": "2617104fc12e40618bb5e4e0d7553672",
                "name": "placement"
            }
        ],
        "user": {
            "password_expires_at": null,
            "domain": {
                "id": "default",
                "name": "Default"
            },
            "id": "0120ff52b86143a4a96c9fbed9cf3fa8",
            "name": "admin"
        },
        "audit_ids": [
            "EggrBIhgThGIrxRXRYFvqg"
        ],
        "issued_at": "2018-10-20T09:16:19.000000Z"
    }
}
```

| Name          | Description   |
|:-------:      | :-------     |
|audit_ids      | 랜덤으로 생성되는 유일한 번호이고, token을 track하기 위해서 사용할 수 있는 Url-Safe한 string이다.|

<br/>

## 2. Project

### - Project List 가져오기

Openstack에서 Project는 tenant(사용자 그룹)을 의미한다.

**Request**

```
URL : {ip:port}/identity/v3/projects
Method : GET
Header : X-Auth-Token : {token}
```

**Response**

```
{
    "links": {
        "self": "http://10.0.2.15/identity/v3/projects",
        "previous": null,
        "next": null
    },
    "projects": [
        {
            "is_domain": false,
            "description": "",
            "links": {
                "self": "http://10.0.2.15/identity/v3/projects/d5ceca181f6b4398aef52183bf4521f1"
            },
            "enabled": true,
            "id": "d5ceca181f6b4398aef52183bf4521f1",
            "parent_id": "default",
            "domain_id": "default",
            "name": "demo"
        }
    ]
}
```

- links

| Name          | Description   |
|:-------:      | :-------     |
|self           | 리소스의 버전을 포함하고 있는 url로 해당 url을 바로 사용해야할 때 사용한다.|
|bookmark       | 영원히 사용되는 url로 기간이 긴 storage에서 사용한다.             |
|alternate      | 리소스를 대체하는 링크를 포함한다.                              |

- pagination

list가 많을 경우 pagination을 하여 subset만 보여주는 경우가 있다.

| Name          | Description   |
|:-------:      | :-------     |
|next           | 이후의 리스트가 있는 링크이다.  |
|previous       | 이전의 리스트가 있는 링크이다. |

<br/>

## 3. Glance

### - Glance List

이미지 리스트를 불러오는 API이다.

**Request**

```
URL : {ip:port}/compute/v2.1/images
Method : GET
Header : X-Auth-Token : {token}
```

**Response**

```
{
    "images": [
        {
            "id": "871b6c50-794e-4381-8da3-636de524b41a",
            "links": [
                {
                    "href": "http://110.10.129.22:8080/compute/v2.1/images/871b6c50-794e-4381-8da3-636de524b41a",
                    "rel": "self"
                },
                {
                    "href": "http://110.10.129.22:8080/compute/images/871b6c50-794e-4381-8da3-636de524b41a",
                    "rel": "bookmark"
                },
                {
                    "href": "http://10.0.2.15/image/images/871b6c50-794e-4381-8da3-636de524b41a",
                    "type": "application/vnd.openstack.image",
                    "rel": "alternate"
                }
            ],
            "name": "cirros-0.3.5-x86_64-disk" # 이미지 이름
        }
    ]
}
```

<br/>

### - Glance Detail List (All Image)

전체 이미지의 상세 정보를 불러오는 API이다.

**Request**

```
URL : {ip:port}/compute/v2.1/images/detail
Method : GET
Header : X-Auth-Token : {token}
```

**Response**

```
{
    "image": {
        "status": "ACTIVE",
        "updated": "2018-08-31T13:20:04Z",
        "links": [
            {
                "href": "http://110.10.129.22:8080/compute/v2.1/images/871b6c50-794e-4381-8da3-636de524b41a",
                "rel": "self"
            },
            {
                "href": "http://110.10.129.22:8080/compute/images/871b6c50-794e-4381-8da3-636de524b41a",
                "rel": "bookmark"
            },
            {
                "href": "http://10.0.2.15/image/images/871b6c50-794e-4381-8da3-636de524b41a",
                "type": "application/vnd.openstack.image",
                "rel": "alternate"
            }
        ],
        "id": "871b6c50-794e-4381-8da3-636de524b41a",
        "OS-EXT-IMG-SIZE:size": 13267968,
        "name": "cirros-0.3.5-x86_64-disk",
        "created": "2018-08-31T13:20:04Z",
        "minDisk": 0,
        "progress": 100,
        "minRam": 0,
        "metadata": {}
    }
}
```

| Name                | Description   |
|:-------:            | :-------     |
|S-EXT-IMG-SIZE:size  | 이미지 사이즈 |
|minDisk              | 이미지를 부팅하기 위해 필요한 disk 공간|
|progress             | 이미지가 저장되는 과정의 Percentage (Active : 100 / Saving : 25 or 50)|
|minRam               | 이미지가 기능을 위한 최소한의 Ram 공간 |
|metadata             | Metadata의 key, value 쌍 |

- status

| Name          | Description   |
|:-------:      | :-------     |
|ACTIVE         | 사용할 수 있는 상태 |
|SAVING         | queue 또는 저장 중인 상태|
|DELETED        | 삭제되었거나, 삭제 중인 상태|
|ERROR          | 에러인 상태|
|UNKNOWN        | 알수 없는 상태|


<br/>

### - Glance Detail Information (Specific Image)

특정 이미지의 상세 정보를 불러오는 API이다.

**Request**

```
URL : {ip:port}/compute/v2.1/images/{image id}
Method : GET
Header : X-Auth-Token : {token}
```

**Response**


```
{
    "image": {
        "status": "ACTIVE",
        "updated": "2018-08-31T13:20:04Z",
        "links": [
            {
                "href": "http://110.10.129.22:8080/compute/v2.1/images/871b6c50-794e-4381-8da3-636de524b41a",
                "rel": "self"
            },
            {
                "href": "http://110.10.129.22:8080/compute/images/871b6c50-794e-4381-8da3-636de524b41a",
                "rel": "bookmark"
            },
            {
                "href": "http://10.0.2.15/image/images/871b6c50-794e-4381-8da3-636de524b41a",
                "type": "application/vnd.openstack.image",
                "rel": "alternate"
            }
        ],
        "id": "871b6c50-794e-4381-8da3-636de524b41a",
        "OS-EXT-IMG-SIZE:size": 13267968,
        "name": "cirros-0.3.5-x86_64-disk",
        "created": "2018-08-31T13:20:04Z",
        "minDisk": 0,
        "progress": 100,
        "minRam": 0,
        "metadata": {}
    }
}
```

<br/>

## 4. Flavor

### - Flavor List

VM의 스펙을 규격화한 정보의 리스트를 보여준다.

**Request**

```
URL : {ip:port}/compute/v2.1/flavors
Method : GET
Header : X-Auth-Token : {token}
```

**Response**

```
{
    "flavors": [
        {
            "id": "1",
            "links": [
                {
                    "href": "http://110.10.129.22:8080/compute/v2.1/flavors/1",
                    "rel": "self"
                },
                {
                    "href": "http://110.10.129.22:8080/compute/flavors/1",
                    "rel": "bookmark"
                }
            ],
            "name": "m1.tiny"
        }, ...
    ]
}
```

<br/>

## 5. Neutron

### - Neutron list

**Request**

```
URL : {ip:port}/v2.0/networks
Method : GET
Header : X-Auth-Token : {token}
```

**Response**

```
{
    "networks": [
        {
            "provider:physical_network": "public",
            "ipv6_address_scope": null,
            "revision_number": 5,
            "port_security_enabled": true,
            "mtu": 1500,
            "id": "1ab2d6c5-3618-47f1-99b3-77fcc1a39294",
            "router:external": true,
            "availability_zone_hints": [],
            "availability_zones": [
                "nova"
            ],
            "ipv4_address_scope": null,
            "shared": false,
            "project_id": "ffc5bce644674cd984f3e0ff67ebe40e",
            "status": "ACTIVE",
            "subnets": [
                "4c8326e8-7a21-4b15-a727-c611b2cf8ff5",
                "d22d8d5e-b9ff-457a-831e-03ee1b34c0ab"
            ],
            "description": "",
            "tags": [],
            "updated_at": "2018-08-31T13:20:40Z",
            "is_default": true,
            "provider:segmentation_id": null,
            "name": "public",
            "admin_state_up": true,
            "tenant_id": "ffc5bce644674cd984f3e0ff67ebe40e",
            "created_at": "2018-08-31T13:20:30Z",
            "provider:network_type": "flat"
        }, ...
    ]
}
```

| Name           | Description   |
|:-------:       | :-------     |
|revision_number | 자원의 revision number를 이용해서 list 결과를 필터링 |
|provider:network_type | flat / vlan / vxlan / gre |
| status | ACTIVE / DOWN / BUILD / ERROR |

<br/>

## Neutron Delete

**Request**

```
URL : {ip:port}/v2.0/networks/{Network id}
Method : DELETE
Header : X-Auth-Token : {token}
```

<br/>

## Neutron Create

**Request**

```
URL : {ip:port}/v2.0/networks
Method : GET
Header : X-Auth-Token : {token}
body :
{
    "network": {
        "admin_state_up": true,
        "name": "net1",
        "provider:network_type": "vlan",
        "provider:physical_network": "public",
        "provider:segmentation_id": 2,
        "qos_policy_id": "6a8454ade84346f59e8d40665f878b2e"
    }
}
```

| Name           | Description   |
|:-------:       | :-------     |
| admin_state_up  | 네트워크의 관리자 상태. (true / false) |
| provider:network_type | flat / vlan / vxlan / gre |
| provider:physical_network | 네트워크가 실행되어야하는 물리적 네트워크 |

**Response**

```
{
    "network": {
        "admin_state_up": true,
        "availability_zone_hints": [],
        "availability_zones": [
            "nova"
        ],
        "created_at": "2016-03-08T20:19:41",
        "dns_domain": "my-domain.org.",
        "id": "4e8e5957-649f-477b-9e5b-f1f75b21c03c",
        "ipv4_address_scope": null,
        "ipv6_address_scope": null,
        "l2_adjacency": true,
        "mtu": 1400,
        "name": "net1",
        "port_security_enabled": true,
        "project_id": "9bacb3c5d39d41a79512987f338cf177",
        "qos_policy_id": "6a8454ade84346f59e8d40665f878b2e",
        "revision_number": 1,
        "router:external": false,
        "shared": false,
        "status": "ACTIVE",
        "subnets": [],
        "tags": ["tag1,tag2"],
        "tenant_id": "9bacb3c5d39d41a79512987f338cf177",
        "updated_at": "2016-03-08T20:19:41",
        "vlan_transparent": false,
        "description": "",
        "is_default": false
    }
}
```

| Name           | Description   |
|:-------:       | :-------     |
| availability_zone_hints  | 네트워크를 위한 후보 가용존   |
| availability_zones | 네트워크를 위한 가용존 |
| l2_adjacency | 	네트워크를 통해서 L2연결이 가능여부를 나타내준다.|
| mtu | maximum transmission unit (MTU) |
| port_security_enabled | 네트워크에서 포트 Security 상태 |
| qos_policy_id | 네트워크에 할당된 QOS 정책 아이디 |

<br/>

## 6. Nova

### - Nova List

인스턴스의 리스트를 볼 수 있는 api이다.

**Request**

```
- URL : {ip:port}/compute/v2.1/servers
- Method : GET
- Header : X-Auth-Token : {token}
```

**Response**

```
{
    "servers": [
        {
            "id": "acdfbbe2-7c64-4643-9211-cf38c604b1ff",
            "links": [
                {
                    "href": "http://110.10.129.22:8080/compute/v2.1/servers/acdfbbe2-7c64-4643-9211-cf38c604b1ff",
                    "rel": "self"
                },
                {
                    "href": "http://110.10.129.22:8080/compute/servers/acdfbbe2-7c64-4643-9211-cf38c604b1ff",
                    "rel": "bookmark"
                }
            ],
            "name": "contributhon_practice"
        }
    ]
}
```

<br/>

### - Nova Detail

인스턴스의 자세한 정보를 볼 수 있는 api이다.

**Request**

```
- URL : {ip:port}/compute/v2.1/servers/detail
- Method : GET
- Header : X-Auth-Token : {token}
```

**Response**

```
{
    "servers": [
        {
            "OS-EXT-STS:task_state": null,
            "addresses": {
                "contributhon-network": [
                    {
                        "OS-EXT-IPS-MAC:mac_addr": "fa:16:3e:15:f1:b7",
                        "version": 4,
                        "addr": "172.31.0.39",
                        "OS-EXT-IPS:type": "fixed"
                    }
                ]
            },
            "links": [
                {
                    "href": "http://110.10.129.22:8080/compute/v2.1/servers/acdfbbe2-7c64-4643-9211-cf38c604b1ff",
                    "rel": "self"
                },
                {
                    "href": "http://110.10.129.22:8080/compute/servers/acdfbbe2-7c64-4643-9211-cf38c604b1ff",
                    "rel": "bookmark"
                }
            ],
            "image": {
                "id": "871b6c50-794e-4381-8da3-636de524b41a",
                "links": [
                    {
                        "href": "http://110.10.129.22:8080/compute/images/871b6c50-794e-4381-8da3-636de524b41a",
                        "rel": "bookmark"
                    }
                ]
            },
            "OS-EXT-STS:vm_state": "active",
            "OS-EXT-SRV-ATTR:instance_name": "instance-00000004",
            "OS-SRV-USG:launched_at": "2018-10-06T12:49:27.000000",
            "flavor": {
                "id": "c1",
                "links": [
                    {
                        "href": "http://110.10.129.22:8080/compute/flavors/c1",
                        "rel": "bookmark"
                    }
                ]
            },
            "id": "acdfbbe2-7c64-4643-9211-cf38c604b1ff",
            "security_groups": [
                {
                    "name": "default"
                }
            ],
            "user_id": "0120ff52b86143a4a96c9fbed9cf3fa8",
            "OS-DCF:diskConfig": "MANUAL",
            "accessIPv4": "",
            "accessIPv6": "",
            "progress": 0,
            "OS-EXT-STS:power_state": 1,
            "OS-EXT-AZ:availability_zone": "nova",
            "config_drive": "",
            "status": "ACTIVE",
            "updated": "2018-10-19T12:03:19Z",
            "hostId": "65a62bb5476213565d51414880d5181943017fb9954b4249ad81e23f",
            "OS-EXT-SRV-ATTR:host": "ubuntu-xenial",
            "OS-SRV-USG:terminated_at": null,
            "key_name": null,
            "OS-EXT-SRV-ATTR:hypervisor_hostname": "ubuntu-xenial",
            "name": "contributhon_practice",
            "created": "2018-10-06T12:49:23Z",
            "tenant_id": "ffc5bce644674cd984f3e0ff67ebe40e",
            "os-extended-volumes:volumes_attached": [],
            "metadata": {}
        }
    ]
}
```

| Name           | Description   |
|:-------:       | :-------     |
|OS-EXT-STS:power_state | 0: NOSTATE / 1: RUNNING / 3: PAUSED / 4: SHUTDOWN / 6: CRASHED / 7: SUSPENDED  |


<br/>

### - Nova Delete

인스 삭제를 하는 API이다.

**Request**

```
- URL : {ip:port}/compute/v2.1/servers/{image id}
- Method : DELETE
- Header : X-Auth-Token : {token}
```

<br/>

### - Nova Create

**Request**

```
URL : {ip:port}/compute/v2.1/servers
Method : POST
Header : X-Auth-Token : {token}
body :
{
    "server": {
        "name": "auto-allocate-network",
        "imageRef": "70a599e0-31e7-49b7-b260-868f441e862b",
        "flavorRef": "http://openstack.example.com/flavors/1",
        "networks": "auto"
    }
}
```

| Name           | Description   |
|:-------:       | :-------     |
|flavorRef | flavor UUID 값 |
|networks | Tenant에 정의된 여러 개의 네트워크 object list|
|imageRef | Image UUID 값 |

**Response**

```
{
    "server": {
        "OS-DCF:diskConfig": "AUTO",
        "adminPass": "6NpUwoz2QDRN",
        "id": "f5dc173b-6804-445a-a6d8-c705dad5b5eb",
        "links": [
            {
                "href": "http://openstack.example.com/v2/6f70656e737461636b20342065766572/servers/f5dc173b-6804-445a-a6d8-c705dad5b5eb",
                "rel": "self"
            },
            {
                "href": "http://openstack.example.com/6f70656e737461636b20342065766572/servers/f5dc173b-6804-445a-a6d8-c705dad5b5eb",
                "rel": "bookmark"
            }
        ],
        "security_groups": [
            {
                "name": "default"
            }
        ]
    }
}
```

- OS-DCF:diskConfig

| Name           | Description   |
|:-------:       | :-------     |
|AUTO	  | flavor disk의 단일 partition 서버를 빌드한다.|
|MANUAL | API는 server 부분 스키마를 사용하고 source image가 있는 file system을 이용하여 빌드한다.|

<br/>

## Openstack App 개발

컨트리뷰톤이라는 주제에 맞춰 API를 연습해보는 것 뿐만 아니라 앱을 만들면 좋겠다는 의견을 취합하여 Openstack App 개발을 진행하게 되었다.

<br/>

### 공통

#### 1. Server List

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_List.jpeg" height="650px"/>

#### - 설명
첫 번째 화면에서는 자신이 등록한 서버의 리스트를 가져오는 화면이 뜬다. 해당 화면을 아래로 내렸다 올리면 페이지가 Reloading된다.

#### 2. Add Server

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_AddServerInfo.png" height="650px"/>

#### - 설명
자신이 생성한 Server의 정보를 입력하면 해당 정보가 저장되며, 목록에 뜨게 된다.

#### 3. Menu

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_menu.png" height="650px"/>

##### - 설명
Navigation 메뉴를 클릭하면 카테고리들이 있는 메뉴가 나오고, Instance, Flavor, Image, Keypair, Network, Router의 정보를 확인할 수 있다. 모든 카테고리에는 CRUD가 제공된다.

<br/>

### Nova

#### 1. Create instance

<img
 src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_ServerInfo.png" height="650px"/>

#### - 설명
인스턴스를 생성하기 위해, 이미지 타입, flavor, Network를 설정한다. 인스턴스를 생성하기 위해서는 많은 옵션이 있지만, 현재는 인스턴스 생성을 위한 가장 기본적인 정보를 이용한다.

#### - api
- URL : ``/servers``
- Method : POST
- body 
	- Server 이름 
	- Image 이름
	- Flavor 종류
	- Network


#### 2. Instance Info

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_Instance.jpeg" height="650px"/>

#### - 설명
서버의 인스턴스 리스트가 나온다.

#### - api
- URL : ``/servers``
- Method : GET

#### 3. Instance Action

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_Instance.png" height="650px"/>

#### - 설명
인스턴스의 상태를 변화시킬 수 있는 페이지이다. 만약 Staus 변화 시 응답받는 로그를 까만 화면 창에 출력해준다.

#### - api
- URL : ``/servers/{server_id}/action``
- Method : POST

<br/>

### Flavor

#### 1. Flavor List

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_flavor.png" height="650px"/>

#### - 설명
미리 정해놓은 인스턴스의 스펙인 Flavor의 리스트이다.

#### - api
- URL : ``/flavors``
- Method : GET

<br/>

### Keypair

#### 1. Keypair List

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_Keypair_List.png" height="650px"/>

#### - 설명
기존의 Keypair의 리스트이다.

#### - api
- URL : ``/os-keypairs``
- Method : GET

#### 2. Keypair Info

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_Keypair.jpeg" height="650px"/>

#### - 설명
Keypair의 상세 정보가 들어있는 페이지이다. 해당 Keypair의 상태를 확인할 수 있고, fingerprint, public_key 등을 확인 가능하다.

#### - api
- URL : ``/os-keypairs/{keypair_name}``
- Method : GET

<br/>

### Image

#### 1. Image List

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_ImageList.png" height="650px"/>

#### - 설명
현재 등록된 이미지 리스트를 가져온다.

#### - api
- URL : ``/v2/images``
- Method : GET

#### 2. Image Info

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_ImageInfo.png" height="650px"/>

#### - 설명
이미지의 자세한 정보들을 볼 수 있는 페이지이다.

#### - api
- URL : ``/v2/images/{image_id}``
- Method : GET

#### 3. Deactive & Reactive

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_ImageActive.png" height="650px"/>

#### - 설명
이미지를 Deactive 시키거나 Reactive 시킬 수 있는데, Deactive시 Reactive로 버튼이 활성화된다.

#### - api
**Deactive** 
- URL : ``/v2/images/{image_id}/actions/deactivate``
- Method : POST

**Reactive**
- URL : ``/v2/images/{image_id}/actions/reactivate``
- Method : POST

#### 4. Create Image

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_Image.jpeg" height="650px"/>

#### - 설명
새로운 이미지를 생성할 수 있다.

#### - api
- URL : ``/v2/images``
- Method : POST
- body 
	- 이미지 이름
	- 이미지 컨테이너 포맷
	- 디스크 포맷

<br/>

### Networking

#### 1. Netwoking List

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_Network.png" height="650px"/>

#### - 설명
현재 존재하는 Network 리스트이다.

#### - api
- URL : ``/v2.0/networks``
- Method : GET

<br/>

### Router

#### 1. Router List

<img src="https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/App_Router.png" height="650px"/>

#### - 설명
현재 존재하는 라우터 리스트이다.

#### - api
- URL : ``/v2.0/routers``
- Method : GET

+) 완성본 동영상 (이미지를 클릭하면 동영상으로 넘어갑니다.)

[![Openstack Application](https://github.com/JaeYeonLee0621/Image/blob/master/Openstack_Contributhon_App/Openstack%20Intro.jpeg)](https://www.youtube.com/watch?v=S4fpy5j_trU&?t=0s "Openstack Application") 
