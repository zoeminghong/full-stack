```
CREATE TABLE T
(    
a_string varchar not null,  
col1 integer  
CONSTRAINT pk PRIMARY KEY (a_string)  
)  
COLUMN_ENCODED_BYTES = none;
```

```
select * from SYSTEM.CATALOG where table_name='T';
```

![image-20190402150657166](assets/image-20190402150657166.png)

```
upsert into t (a_string,col1)values('gejx',1);
```

![image-20190402150725617](assets/image-20190402150725617.png)

```
CREATE TABLE T
(    
a_string varchar not null,  
col1 integer  
CONSTRAINT pk PRIMARY KEY (a_string)  
)  
COLUMN_ENCODED_BYTES = 0;
```

![image-20190402150725617](assets/image-20190402150725617.png)

