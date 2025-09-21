# 📄 Metadata Registration & Search Configuration in DSpace 8.x  
## 🧾 Adding Metadata Fields: `dc.judge.name`

This document outlines the steps to register new metadata fields in DSpace 7.x and configure them for:
- Solr indexing
- Faceted search
- Submission forms
- Browse and sort options

---

## 🧱 Step 1: Register Metadata Fields

### 🔧 File:


[dspace]/config/registries/dublin-core-types.xml
```

### ➕ Add:
```xml
<!-- Case Number -->

    <dc-type>
       <schema>dc</schema>
       <element>judge</element>
       <qualifier>name</qualifier>
       <scope_note>Name of the Judge</scope_note>
    </dc-type>

```

### ✅ Why:
To register these metadata fields in the internal DSpace metadata registry.

---

## 🔎 Step 2: Configure Solr Search Filters

### 🔧 File:
```
[dspace]/config/spring/discovery.xml
```

### ➕ Under `<property name="searchFilters">`:
```xml
<ref bean="searchFilterJudgeName" />
```

### ➕ Define the filter beans:
```xml

<bean id="searchFilterJudgeName" class="org.dspace.discovery.configuration.DiscoverySearchFilterFacet">
    <property name="indexFieldName" value="dc_judge_name"/>
    <property name="metadataFields">
        <list>
            <value>dc.judge.name</value>
        </list>
    </property>
    <property name="facetLimit" value="5"/>
    <property name="type" value="text"/>
</bean>
```

### ✅ Why:
To enable faceted filtering in the sidebar during discovery searches.

---

## 🔁 Step 3: Add Sort Options for New Fields

### 🔧 File:
```
[dspace]/config/spring/discovery.xml
```

### ➕ Under `<property name="searchSortConfiguration">`:
```xml
<ref bean="searchFilterJudgeName" />
```

(Define beans for each if required.)

---

## 📇 Step 4: Include Metadata in Submission Form

### 🔧 File:
```
[dspace]/config/submission-forms.xml
```

### ➕ Add inside `<row>` blocks:
```xml
<row>
    <field>
        <dc-schema>dc</dc-schema>
        <dc-element>judge</dc-element>
        <dc-qualifier>name</dc-qualifier>
        <label>Judge name</label>
        <input-type>onebox</input-type>
        <hint>Enter the Judge name</hint>
        <required>if you write something in this tag then it is mandatory field</required>
    </field>
</row>
```

### ✅ Why:
To make these metadata fields appear in the submission and edit item form.

---

## 🔍 Step 5: Configure Solr Indexing

### 🔧 File:
```
[dspace]/solr/search/conf/schema.xml
```

### ➕ Add Solr fields:
```xml
    <field name="dc.judge.name" type="text" indexed="true" stored="true" multiValued="false" /> 
    <copyField source="dc.judge.name" dest="dc_judge_name"/>  

```

### ✅ Why:
To enable proper indexing and copying of metadata for search filtering.

---

## 📂 Step 6: Add Browse Index

### 🔧 File:
```
[dspace]/config/dspace.cfg
```

### ➕ Add:
```properties
webui.browse.index.4 = dc_judge_name:metadata:dc.judge.name:text
```

### ✅ Why:
To allow users to browse items using a combination of case metadata.

---

## 🔄 Step 7: Rebuild, Reindex, and Restart Services

### 📌 Run:
```bash
# Export metadata (if necessary) in [dspace]
sudo ./dspace registry-loader -metadata ../config/registries/dublin-core-types.xml
# Reindex Discovery
sudo ./dspace index-discovery -b
# Restart DSpace services
sudo ./startup.sh
sudo ./shutdown.sh


sudo ./solr start -p 8983 -force
sudo ./solr stop -p 8983 -force

```

---

## ✅ Final Outcome

By completing these steps, you will:
- Register `dc.judge.name`
- Enable search filtering and browse indexing
- Allow submission/editing via DSpace UI
- Power discovery, analytics, and automation features with structured case metadata


---

> **Author:** Reshwanth  
> **Last Updated:** April 21, 2025  
> **Target DSpace Version:** 7.6.2+
```

