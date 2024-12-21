# Architecture and Data Flow Diagram

```plaintext
                         +---------------------------+
                         |     SQL Server (Prod)     |
                         |  - Sensitive Data Source  |
                         |  - Generates JSON Output  |
                         +---------------------------+
                                     |
                                     | (Pull Data)
                                     v
      +-------------------------------+-------------------------------+
      |              Dev Server (Node.js + SQLite)                   |
      |  - Pulls data from SQL Server                                |
      |  - Encrypts data into JSON files                             |
      |  - Stores JSON in SQLite for staging and testing             |
      |  - Pushes JSON files to Prod and Public Servers              |
      +-------------------------------+-------------------------------+
                      | (Push Encrypted JSON)              | (Push Encrypted JSON)
                      v                                     v
      +-----------------------+                 +-----------------------+
      |     Prod Server       |                 |   Public Server       |
      |  (Node.js Backend)    |                 |  (Static Hosting)     |
      |  - Hosts secure pages |                 |  - Serves static pages|
      |  - Reads JSON files   |                 |  - Reads JSON files   |
      +-----------------------+                 +-----------------------+
                                     |
                                     | (Access Encryption Keys & Secrets)
                                     v
                         +---------------------------+
                         |    Secrets Manager        |
                         |  - Stores API Keys        |
                         |  - Manages Encryption Keys|
                         +---------------------------+
