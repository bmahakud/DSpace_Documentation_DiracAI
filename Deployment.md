<!-- step 1: make a drive link of updated backend and frontend zip file seperately
step 2: now make a new folder inside dspace in the server with the <month>_<date> (eg:july_7th)
step 3: now create few folder inside this new folder source , root , f-root,
step 4: now extract the zip files named frontend , backend inside source folder (optionla delete the zip folder if required)
step 5: now open terminal in the source location and type code . -> this opens the vs code from that directory
step 6: lets fix backend now
    step 6.1: navigate to backend/dspace/config/dspace.cfg
        step 6.1.1: chnage the dspace.dir to newly created root path (eg:/home/dspace/dspace/july_7th/root)
        step 6.1.2: change the ip address of bakend serving from "localhost:8080/server" -> "http:<ip>:8080/server"
        step 6.1.3: change the ip addrees of where frontend is serving from "localhost:4000" -> "http:<ip>:4000"
        step 6.1.4: change the ip address of solr from "localhost:8983" -> "http:<ip>:8983"
        step 6.1.5: change the ip address of postgres from "localhost:5432" -> "http:<op>:5432"
        step 6.1.6: change the db user and db password to anthem 
    step 6.2: now open terminal navigate to source/backend directory
        step 6.2.1: run '''mvn clean package'''
        step 6.2.2: after build successfull the run '''cd dspace/target/dspace-intsaller/
        step 6.2.3: now run '''ant fresh_install'''
    step 6.3: if you have changed the file in backend/dspace/....... then follow 6.4 else follow 6.5
    step 6.7: navigate to root/solr copy all the folders and paste in servver you cane refer the README.md file for that 
        step 6.7.1: register the metadata 
        step 6.7.2: migrate the date
        step 6.7.3: update the discovery
            note : these steps were explained in README.md file
    step 6.8: now navigate to root/webapps and copy the server folder and paste in server/apache<verion>/webapps delete the sever folder if any existed
        step 6.8.1: now restart the tomcat server 
    step 6.9: now copy the assetstore folder from the previous deployement (eg: july_2nd/root/assertstore) and paste in the root/


step 7: lets fix frontend now 
    step 7.1: navigate to frontend and open terminal
    step 7.2: now change the variable in the frontend code from development -> production
    step 7.3: now run npm start in frontend/ directory
    step 7.4: now copy the config.prod.yml from previos deployment(eg: junly_2nd/f-root/config.prod.yml) and paste in f-root/
    step 7.5: after build succes copy the dist folder from frontend/dist and paste in f-root/
    step 7.6: stop frontend server if it is running and navigate to f-root firectory
    step 7.7: run command '''node/dist/server/main.js'''


Frequent errors check list:
    New tables or change in table (need to migrate it first)
    permission to access those tables or coluuns (dspace user and anthem user needs to able to grant the permission to change the tables)
    File permissions issue contact me need to change the code
     -->



---

# Deployment Steps

### Step 1

Make a drive link of updated **backend** and **frontend** zip file separately.

---

### Step 2

Now make a new folder inside `dspace` in the server with the `<month>_<date>`
Example:

```
july_7th
```

---

### Step 3

Now create few folders inside this new folder:

* `source`
* `root`
* `f-root`

---

### Step 4

Now extract the zip files named **frontend**, **backend** inside `source` folder
(Optional: delete the zip folder if required)

---

### Step 5

Now open terminal in the `source` location and type:

```bash
code .
```

→ this opens VS Code from that directory

---

### Step 6: Lets fix backend now

#### Step 6.1

Navigate to:

```
backend/dspace/config/dspace.cfg
```

* **Step 6.1.1**: change the `dspace.dir` to newly created root path

  ```
  /home/dspace/dspace/july_7th/root
  ```
* **Step 6.1.2**: change the ip address of backend serving

  ```
  "localhost:8080/server" -> "http:<ip>:8080/server"
  ```
* **Step 6.1.3**: change the ip address of where frontend is serving

  ```
  "localhost:4000" -> "http:<ip>:4000"
  ```
* **Step 6.1.4**: change the ip address of Solr

  ```
  "localhost:8983" -> "http:<ip>:8983"
  ```
* **Step 6.1.5**: change the ip address of Postgres

  ```
  "localhost:5432" -> "http:<ip>:5432"
  ```
* **Step 6.1.6**: change the DB user and DB password to `anthem`

---

#### Step 6.2

Now open terminal and navigate to `source/backend` directory

* **Step 6.2.1**

  ```bash
  mvn clean package
  ```
* **Step 6.2.2**
  After build successful then run:

  ```bash
  cd dspace/target/dspace-installer/
  ```
* **Step 6.2.3**

  ```bash
  ant fresh_install
  ```

---

#### Step 6.3

If you have changed the file in `backend/dspace/...` then follow **6.4** else follow **6.5**

---

#### Step 6.7

Navigate to `root/solr` copy all the folders and paste in server (you can refer the `README.md` file for that)

* **Step 6.7.1**: register the metadata
* **Step 6.7.2**: migrate the data
* **Step 6.7.3**: update the discovery

> Note: these steps were explained in `README.md` file

---

#### Step 6.8

Now navigate to `root/webapps` and copy the `server` folder and paste in:

```
server/apache<version>/webapps
```

Delete the `server` folder if any existed.

* **Step 6.8.1**: now restart the Tomcat server

---

#### Step 6.9

Now copy the `assetstore` folder from the previous deployment, e.g.:

```
july_2nd/root/assetstore → july_7th/root/assetstore
```

---

### Step 7: Lets fix frontend now

#### Step 7.1

Navigate to `frontend` and open terminal

#### Step 7.2

Now change the variable in the frontend code from **development** → **production**

#### Step 7.3

Run:

```bash
npm start
```

in `frontend/` directory

#### Step 7.4

Now copy the `config.prod.yml` from previous deployment, e.g.:

```
july_2nd/f-root/config.prod.yml → july_7th/f-root/config.prod.yml
```

#### Step 7.5

After build success copy the `dist` folder from:

```
frontend/dist → f-root/
```

#### Step 7.6

Stop frontend server if it is running and navigate to `f-root` directory

#### Step 7.7

Run command:

```bash
node dist/server/main.js
```

---

### Frequent Errors Checklist

* New tables or change in table (need to migrate it first)
* Permission to access those tables or columns (dspace user and anthem user needs to be able to grant the permission to change the tables)
* File permissions issue → contact me, need to change the code

---
