# Proyecto-I-AdminTecnologia
Proyecto I
Consiste en implementar una infraestructura de comercio electrónico. La base de datos a utilizar es
MySQL, MariaDB, MSSQL Server, PostgreSQL u Oracle y debe tener replicación, es decir maestro –
esclavo, donde solo en el nodo maestro se podrán realizar las instrucciones DML y DDL. El esclavo
únicamente debe ser de consulta. La aplicación de e-commerce implementada debe apuntar al nodo
maestro

Por otro lado debe implementar una plataforma para KPI y/o tableros de control. Es recomendable usar
Grafana para ello, sin embargo queda libre para proponer. La fuente de datos para el tablero de control
debe ser el nodo esclavo de la base de datos. Si considera realizar una trasnformación de datos es decir
ETL lo puede realizar.
El diseño del tablero o KPI queda abierto para que haga su mejor propuesta, entre más intuitivo y
detallado esté mejor calificado será de lo contrario puede disminuir. Como mínimo deben estar en red
tres (puede ser más) hosts (computadoras o VPS o virtuales, etc.); maestro, esclavo y servidor de
aplicaciones web. Para conectarlas se deben pegar a una VPN para el día de la calificación se utilizará
OpenVPN y los datos de conexión se los proporcionarán en ese momento. La computadora del
evaluador será el cuarto (si solo se implementaron tres instancias).
Uno de los retos al presentar es hacer la configuración a la VPN y las modificaciones a su proyecto
para que tome las IPs que les proporcione el mismo, por tal razón se le recomienda hacer pruebas de
ello también.
Para la evaluación se debe proporcionar las URL tanto para el e-commerce como la del tablero de
control, usando las IPs asignada por la VPN.
Todo debe correr sobre sistemas operativos GNU/Linux, BSD o Unix y si la aplicación elegida necesita
SO Windows será la única excepción. Tambien sobre el protocolo HTTPS, puede usar OpenSSL para
generar el certificado o Let’s Encrypt si lo implementa en una VPS.
