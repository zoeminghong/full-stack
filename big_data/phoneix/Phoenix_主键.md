# Phoenix 主键

## 联合组件

```sql
create table "test_keys" ("createTime" date not null, "wara_house" integer not null constraint pk primary key("createTime", "wara_house"));
```

