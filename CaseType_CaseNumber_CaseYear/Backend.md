# 📄 Metadata Registration & Search Configuration in DSpace 8.x  
## 🧾 Adding Metadata Fields: `dc.case.number`, `dc.caseyear`, `dc.casetype`

This document outlines the steps to register new metadata fields in DSpace 7.x and configure them for:
- Solr indexing
- Faceted search
- Submission forms
- Browse and sort options

---

## 🧱 Step 1: Register Metadata Fields

### 🔧 File:
```
[dspace]/config/registries/dublin-core-types.xml
```

### ➕ Add: inside dspace-dc-types tag
```xml
<!-- Case Number -->
<dc-type>
    <schema>dc</schema>
    <element>case</element>
    <qualifier>number</qualifier>
    <scope_note></scope_note>
</dc-type>

<!-- Case Year -->
<dc-type>
    <schema>dc</schema>
    <element>caseyear</element>
    <scope_note>Case Year</scope_note>
</dc-type>

<!-- Case Type -->
<dc-type>
    <schema>dc</schema>
    <element>casetype</element>
    <scope_note>Case Type</scope_note>
</dc-type>
```

### ✅ Why:
To register these metadata fields in the internal DSpace metadata registry.

---

## 🔎 Step 2: Configure Solr Search Filters

### 🔧 File:
```
[dspace]/config/spring/api/discovery.xml
```

### ➕ Under `<property name="searchFilters">`: inside `bean id="defaultConfiguration"`
```xml
<ref bean="searchFilterCaseNumber" />
<ref bean="searchFilterCaseType" />
<ref bean="searchFilterCaseYear" />
<ref bean="searchFilterCaseNumberType" />
```

### ➕ Define the filter beans:
```xml
<bean id="searchFilterCaseNumber" class="org.dspace.discovery.configuration.DiscoverySearchFilterFacet">
    <property name="indexFieldName" value="dc_case_number"/>
    <property name="metadataFields">
        <list><value>dc.case.number</value></list>
    </property>
    <property name="facetLimit" value="5"/>
</bean>

<bean id="searchFilterCaseType" class="org.dspace.discovery.configuration.DiscoverySearchFilterFacet">
    <property name="indexFieldName" value="dc_case_type"/>
    <property name="metadataFields">
        <list><value>dc.casetype</value></list>
    </property>
    <property name="facetLimit" value="5"/>
</bean>

<bean id="searchFilterCaseYear" class="org.dspace.discovery.configuration.DiscoverySearchFilterFacet">
    <property name="indexFieldName" value="dc_case_year"/>
    <property name="metadataFields">
        <list><value>dc.caseyear</value></list>
    </property>
    <property name="facetLimit" value="5"/>
</bean>

<bean id="searchFilterCaseNumberType" class="org.dspace.discovery.configuration.DiscoverySearchFilterFacet">
    <property name="indexFieldName" value="dc_case_numberType"/>
    <property name="metadataFields">
        <list>
            <value>dc.case.number</value>
            <value>dc.casetype</value>
            <value>dc.caseyear</value>
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
[dspace]/config/spring/api/discovery.xml  
```

### ➕ Under `<property name="searchSortConfiguration">`: inside `bean id="defaultConfiguration"`
```xml
<ref bean="sortByCaseNumber" />
<ref bean="sortByCaseType"/>
<ref bean="sortByCaseYear"/>
```

(Define beans for each if required.)

---

## 📇 Step 4: Include Metadata in Submission Form

### 🔧 File:
```
[dspace]/config/submission-forms.xml
```

### ➕ Add inside `<row>` blocks: inside `<form name="bitstream-metadata">`

```xml
<row>
    <field>
        <dc-schema>dc</dc-schema>
        <dc-element>casetype</dc-element>
        <repeatable>true</repeatable>
        <label>Case Type</label>
        <input-type value-pairs-name="common_types">dropdown</input-type>
        <hint>Specify the type of legal case.</hint>
        <required>Yes</required>
    </field>
</row>

<row>
    <field>
        <dc-schema>dc</dc-schema>
        <dc-element>case</dc-element>
        <dc-qualifier>number</dc-qualifier>
        <repeatable>true</repeatable>
        <label>Case Number</label>
        <input-type>onebox</input-type>
        <required>Yes</required>
    </field>
</row>

<row>
    <field>
        <dc-schema>dc</dc-schema>
        <dc-element>caseyear</dc-element>
        <repeatable>true</repeatable>
        <label>Case Year</label>
        <input-type>onebox</input-type>
        <required>Yes</required>
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

### ➕ Add Solr fields: inside last `<fields> </fields>` tag
```xml
<field name="dc.case.number" type="string" indexed="true" stored="true" multiValued="false"/>
<field name="dc.casetype" type="string" indexed="true" stored="true" multiValued="false"/>
<field name="dc.caseyear" type="string" indexed="true" stored="true" multiValued="false"/>

<copyField source="dc.case.number" dest="dc_case_number"/>
<copyField source="dc.casetype" dest="dc_case_type"/>
<copyField source="dc.caseyear" dest="dc_case_year"/>
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
webui.browse.index.2 = dc_case_type:metadata:dc.case.type\,dc.case.number\,dc.case.year:text
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
- Register `dc.case.number`, `dc.caseyear`, `dc.casetype`
- Enable search filtering and browse indexing
- Allow submission/editing via DSpace UI
- Power discovery, analytics, and automation features with structured case metadata

---



> **Author:** Reshwanth  
> **Last Updated:** April 21, 2025  
> **Target DSpace Version:** 8.x+
```

