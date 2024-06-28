[![FIWARE Banner](https://fiware.github.io/tutorials.Application-Mashup/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Visualization](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/visualization.svg)](https://www.fiware.org/developers/catalogue/)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Application-Mashup.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware-wirecloud)

This tutorial is an introduction to [FIWARE Wirecloud](https://Wirecloud.rtfd.io) - a generic enabler visualization tool
which allows end users without programming skills to create web applications and dashboards to visualize their NGSI
data. The tutorial explains how to create a Wirecloud workspace and upload widget to visualise the data. Once the
widgets are configured the data is displayed on screen

The tutorial demonstrates examples of interactions using the Wirecloud GUI only. No programming is involved within the
tutorial itself, as Wirecloud is designed to be usable by all type of user, even those with limited programming skills.
However the commentary continues to reference various programming principles and standard concepts common to all FIWARE
architectures.

# Start-Up

**NGSI-v2** offers JSON based interoperability used in individual Smart Systems. To run this tutorial with **NGSI-v2**, use the `NGSI-v2` branch.

```console
git clone https://github.com/FIWARE/tutorials.Administrating-XACML.git
cd tutorials.Administrating-XACML
git checkout NGSI-v2

./services create
./services start
```

| [![NGSI v2](https://img.shields.io/badge/NGSI-v2-5dc0cf.svg)](https://fiware-ges.github.io/orion/api/v2/stable/) | :books: [Documentation](https://github.com/FIWARE/tutorials.Application-Mashup/tree/NGSI-v2) | ![](https://img.shields.io/github/last-commit/fiware/tutorials.Application-Mashup/NGSI-v2)
| --- | --- | --- |

---

## License

[MIT](LICENSE) © 2019-2024 FIWARE Foundation e.V.

