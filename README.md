[![FIWARE Banner](https://jason-fox.github.io/tutorials.Application-Mashup/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Visualization](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/visualization.svg)](https://www.fiware.org/developers/catalogue/)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Application-Mashup.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/wirecloud.svg)](https://stackoverflow.com/questions/tagged/fiware-wirecloud)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

## Contents

<details>
<summary><strong>Details</strong></summary>

-   [Visualizing NGSI Data using a Mashup](#visualizing-ngsi-data-using-a-mashup)
-   [Prerequisites](#prerequisites)
    -   [Docker](#docker)
    -   [Cygwin](#cygwin)
-   [Architecture](#architecture)
    -   [Wirecloud Configuration](#wirecloud-configuration)
-   [Start Up](#start-up)
    -   [Log in](#log-in)
-   [Adding Resources to Wirecloud](#adding-resources-to-wirecloud)
    -   [Upload Widgets](#upload-widgets)
    -   [Creating a Workspace](#creating-a-workspace)
-   [Creating Application Mashups](#creating-application-mashups)
    -   [Creating a simple Mashup](#creating-a-simple-mashup)
        -   [Selecting Widgets](#selecting-widgets)
        -   [NSGI Browser Widget](#nsgi-browser-widget)
    -   [Combining Multiple Widgets within a Mashup](#combining-multiple-widgets-within-a-mashup)
        -   [Selecting Widgets and Operators](#selecting-widgets-and-operators)
        -   [NGSI Source Operator](#ngsi-source-operator)
        -   [NGSI Entity to POI Operator](#ngsi-entity-to-poi-operator)
        -   [Open Layers Map Widget](#open-layers-map-widget)

</details>

# Visualizing NGSI Data using a Mashup

TBD

# Prerequisites

## Docker

To keep things simple all components will be run using [Docker](https://www.docker.com). **Docker** is a container
technology which allows to different components isolated into their respective environments.

-   To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
-   To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
-   To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A
[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.Identity-Management/master/docker-compose.yml) is used
configure the required services for the application. This means all container services can be brought up in a single
command. Docker Compose is installed by default as part of Docker for Windows and Docker for Mac, however Linux users
will need to follow the instructions found [here](https://docs.docker.com/compose/install/)

## Cygwin

We will start up our services using a simple bash script. Windows users should download [cygwin](http://www.cygwin.com/)
to provide a command-line functionality similar to a Linux distribution on Windows.

# Architecture

TBD

Therefore the overall architecture will consist of the following elements:

-   The FIWARE [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which will receive requests using
    [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2)
-   The FIWARE [IoT Agent for UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/) which will receive
    southbound requests using [NGSI](https://fiware.github.io/specifications/OpenAPI/ngsiv2) and convert them to
    [UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
    commands for the devices
-   The FIWARE [Keyrock](https://fiware-idm.readthedocs.io/en/latest/) Identity Management System
    -   Used by both the **Stock Management System** and **Wirecloud**
-   FIWARE [Wirecloud](https://wirecloud.readthedocs.io/en/stable/) an application mashup tool for displaying NGSI
    entities
-   Three databases

    -   A [PostgreSQL](https://www.postgresql.org/) database :
        -   Used by **Wirecloud** to hold mashup state
    -   A [MySQL](https://www.mysql.com/) database :
        -   Used by **Keyrock** to persist user identities, applications, roles and permissions
    -   A [MongoDB](https://www.mongodb.com/) database:
        -   Used by the **Orion Context Broker** to hold context data information such as data entities, subscriptions
            and registrations
        -   Used by the **IoT Agent** to hold device information such as device URLs and Keys

-   The **Stock Management Frontend** does the following:
    -   Displays store information
    -   Shows which products can be bought at each store
    -   Allows users to "buy" products and reduce the stock count.
-   A webserver acting as set of [dummy IoT devices](https://github.com/FIWARE/tutorials.IoT-Sensors) using the
    [UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
    protocol running over HTTP - access to certain resources is restricted.
-   Three additional microservices are used by **Wirecloud**:
    -   [Memcache](memcached.org), a general-purpose distributed memory caching system.
    -   [ElasticSearch](https://elastic.co/products/elasticsearch), a full-text search engine
    -   [NGSI Proxy](https://github.com/conwetlab/ngsi-proxy), a server that is capable of redirecting **Orion**
        notifications to web pages.

Since all interactions between the elements are initiated by HTTP requests, the entities can be containerized and run
from exposed ports.

![](https://jason-fox.github.io/tutorials.Application-Mashup/img/architecture.png)

The specific architecture of each section of the tutorial is discussed below.

## Wirecloud Configuration

```yaml
image: fiware/wirecloud
        container_name: fiware-wirecloud
        hostname: wirecloud
        ports:
            - "8000:8000"
        networks:
          default:
            ipv4_address: 172.18.1.10

        restart: always
        depends_on:
            - keyrock
            - elasticsearch
            - memcached
            - postgres-db
        environment:
            - DEBUG=True
            - DEFAULT_THEME=wirecloud.defaulttheme
            - DB_HOST=postgres-db
            - DB_PASSWORD=wirepass
            - FORWARDED_ALLOW_IPS=*
            - ELASTICSEARCH2_URL=http://elasticsearch:9200/
            - MEMCACHED_LOCATION=memcached:11211
            - FIWARE_IDM_URL=http://localhost:3005
            - FIWARE_IDM_SERVER=http://172.18.1.5:3005
            - SOCIAL_AUTH_FIWARE_KEY=wirecloud-dckr-site-0000-00000000000
            - SOCIAL_AUTH_FIWARE_SECRET=wirecloud-docker-000000-clientsecret
        volumes:
            - wirecloud-data:/opt/wirecloud_instance/data
            - wirecloud-static:/var/www/static
```

The `wirecloud` container is a web application server listening on a single port:

-   Port `8000` has been exposed for HTTP traffic so we can display the web page

The `wirecloud` container is connecting to **Keyrock** and is driven by environment variables as shown:

| Key                       | Value                                  | Description                                                                          |
| ------------------------- | -------------------------------------- | ------------------------------------------------------------------------------------ |
| DEFAULT_THEME             | `wirecloud.defaulttheme`               | Which Wirecloud theme to display                                                     |
| DB_HOST                   | `postgres-db`                          | The name of the Wirecloud database                                                   |
| DB_PASSWORD               | `wirepass`                             | The password for the Wirecloud database - this should be protected by Docker Secrets |
| FORWARDED_ALLOW_IPS       | `*`                                    |                                                                                      |
| ELASTICSEARCH2_URL        | `http://elasticsearch:9200/`           | The location the ElasticSearch service is listening on                               |
| MEMCACHED_LOCATION        | `memcached:11211`                      | The location the Memcahe service is listening on                                     |
| FIWARE_IDM_URL            | `http://localhost:3005`                | The URL of **Keyrock** used to display the login screen                              |
| FIWARE_IDM_SERVER         | `http://172.18.1.5:3005`               | The URL of **Keyrock** used for OAuth2 Authentication                                |
| SOCIAL_AUTH_FIWARE_KEY    | `wirecloud-dckr-site-0000-00000000000` | The Client ID defined by **Keyrock** for **Wirecloud**                               |
| SOCIAL_AUTH_FIWARE_SECRET | `wirecloud-docker-000000-clientsecret` | The Client Secret defined by **Keyrock** for **Wirecloud**                           |

# Start Up

To start the installation, do the following:

```console
git clone git@github.com:FIWARE/tutorials.Application-Mashup.git
cd tutorials.Application-Mashup

./services create
```

> **Note** The initial creation of Docker images can take up to three minutes

Thereafter, all services can be initialized from the command-line by running the
[services](https://github.com/FIWARE/tutorials.Application-Mashup/blob/master/services) Bash script provided within the
repository:

```console
./services start
```

> :information_source: **Note:** If you want to clean up and start over again you can do so with the following command:
>
> ```console
> ./services stop
> ```

## Log in

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/login.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/back-button.png)

# Adding Resources to Wirecloud

## Upload Widgets

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/my-resources-button.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/upload-button.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/upload-widgets.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/upload-components-list.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/my-resources.png)

## Creating a Workspace

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/new-workspace.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/create-workspace.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/selecting-a-workspace.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/workspace.png)

# Creating Application Mashups

## Creating a simple Mashup

### Selecting Widgets

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/wiring-view.png)

### NSGI Browser Widget

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/ngsi-browser-widget.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/ngsi-browser-wiring.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/ngsi-browser-settings.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/devices-lamp-on.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/ngsi-browser-ui.png)

## Combining Multiple Widgets within a Mashup

### Selecting Widgets and Operators

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/add-components.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/operators-list.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/widgets-list.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/components-widgets.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/osm-unwired.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/osm-wired.png)

### NGSI Source Operator

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/ngsi-source-wiring.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/ngsi-source-settings.png)

### NGSI Entity to POI Operator

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/ngsi-to-poi-wiring.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/ngsi-to-poi-settings.png)

### Open Layers Map Widget

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/osm-wiring.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/osm-settings.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/osm-map-result.png)

-   ![](https://jason-fox.github.io/tutorials.Application-Mashup/img/osm-map-on-click.png)

# Next Steps

Want to learn how to add more complexity to your application by adding advanced features? You can find out by reading
the other [tutorials in this series](https://fiware-tutorials.rtfd.io)

---

## License

[MIT](LICENSE) © 2019 FIWARE Foundation e.V.
