[![FIWARE Banner](https://jason-fox.github.io/tutorials.Application-Mashup/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Visualization](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/visualization.svg)](https://www.fiware.org/developers/catalogue/)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Application-Mashup.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/wirecloud.svg)](https://stackoverflow.com/questions/tagged/fiware-wirecloud)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

This tutorial is an introduction to [FIWARE Wirecloud](https://Wirecloud.rtfd.io) - a generic enabler visualization tool
which allows end users without programming skills to create web applications and dashboards to visualize their NGSI
data. The tutorial explains how to create a Wirecloud workspace and upload widget to visualise the data. Once the
widgets are configured the data is displayed on screen

The tutorial demonstrates examples of interactions using the Wirecloud GUI only. No programming is involved within the
tutorial itself, as Wirecloud is designed to be usable by all type of user, even those with limited programming skills.
However the commentary continues to reference various programming principles and standard concepts common to all FIWARE
architectures.

Additional materials covering how to develop and create your own widgets will be the subject of a later tutorial.

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

> "One picture is worth a thousand words."
>
> — Fred R. Barnard (Printers' Ink)

As a Smart Solution evolves, it is necessary to be able to analyse and understand the current system context, so that
appropriate decisions can be made. An obvious first step of data analysis is to display context data on screen. At this
stage it is not necessarily known which context data is relevant and how it would be best displayed to the end user.
Flexible rapid prototyping is required to be able to refine, enhance and manipulate the data, display and tweak the
visualizations, and add further information as necessary.

An **application mashup** is a web application which uses content from multiple sources to display a single new service
with a graphical interface. Most mashups are visual and interactive by design, and many are short-lived representations
which are only required to help analyse a single problem.

The [FIWARE Wirecloud](https://Wirecloud.rtfd.io) Generic Enabler is a tool which helps users to rapidly generate new
application mashups based on NGSI and other data sources. To speed up development, the Wirecloud architecture has been
defined to split mashup operations into a series of simple reusable tasks (widgets and operators). Each task has
well-defined input and output interfaces, and the Wirecloud UI allows mashup creators to wire up a series of tasks into
a complex chain of data processing and visualization events.

Broadly speaking application mashup tasks can be split into four categories:

-   **Data sources**: These are operators that provide information for consumption elsewhere. For example, an operator
    that retrieves some type of information from an NGSI web service.
-   **Data targets**: The reverse of the above. These operators push information out to an external microservice
-   **Data transformers**: This type of operator manipulates data in order to make it usable by other tasks further down
    the chain within the Wirecloud ecosystem. For example, transposing form list values or renaming attributes to align
    with an input interface downstream
-   **Visual Components**: Combinations of HTML and JavaScript which display data on a browser online. Within Wirecloud,
    visual components are known as widgets.

The overall aim of Wirecloud is to allow someone without a programming background to be able to create data
visualizations using a drag-and-drop interface. A wide range of existing open-source
[Wirecloud Widgets and Operators](https://wirecloud.readthedocs.io/en/stable/widgets/) are already available and can be
used to create complex visualizations.

The existing [widget and operator set](https://wirecloud.readthedocs.io/en/stable/widgets/) covers a wide range of
scenarios, but can be complemented by your own additional widgets. A background in JavaScript and HTML is necessary in
this case. Creating your own widgets will be the subject of a subsequent tutorial.

Once a mashup has been wired up and created it can be also be shared wholesale with end users.

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

This application adds the Wirecloud Application Mashup into the existing Stock Management and Sensors-based application
created in [previous tutorials](https://github.com/FIWARE/tutorials.IoT-Agent/). The aim of the tutorial is to be able
to monitor devices and wire-up a simple supermarket finder. This monitoring tool mashup will be able to duplicate and
replace much of the visualisation functionality already found in the tutorial application itself (which is written in
Jade Node.JS and JavaScript). The aim is to create an equivalent application without resorting to writing a line of
code.

The Users in Wirecloud have been created using the standard
[identity management](https://github.com/FIWARE/tutorials.Identity-Management/) component. Overall the system makes make
use of four FIWARE components - the [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/),the
[IoT Agent for UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/), the
[Keyrock](https://fiware-idm.readthedocs.io/en/latest/) Identity Manager and the newly integrated
[Wirecloud](https://wirecloud.readthedocs.io/en/stable/) application mashup tool. Usage of the Orion Context Broker is
sufficient for an application to qualify as _“Powered by FIWARE”_.

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
            - FIWARE_IDM_PUBLIC_URL=http://localhost:3005
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
| FIWARE_IDM_PUBLIC_URL     | `http://localhost:3005`                | The URL of **Keyrock** used to display the login screen                              |
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
