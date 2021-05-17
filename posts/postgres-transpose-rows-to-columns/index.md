# Postgres Transpose Rows to Columns


<!--more-->

延續上一篇文章 [Rbac1 Design ｜ KaiChu](https://kaichu.io/posts/rbac1-design/), 我們透過 Postgres 中的 CTE (common table expression) recursive query 來查詢有繼承 Role 的有對應的權限

### user_id permission

| user\_id | key |
| :--- | :--- |
| 87gb8fKJHGxh2Pz\_Gk\_R2 | create:devops |
| 87gb8fKJHGxh2Pz\_Gk\_R2 | create:rbac |
| 87gb8fKJHGxh2Pz\_Gk\_R2 | create:users |
| 87gb8fKJHGxh2Pz\_Gk\_R2 | delete:devops |
| 87gb8fKJHGxh2Pz\_Gk\_R2 | delete:rbac |
| 87gb8fKJHGxh2Pz\_Gk\_R2 | delete:users |
| 87gb8fKJHGxh2Pz\_Gk\_R2 | read:devops |
| 87gb8fKJHGxh2Pz\_Gk\_R2 | read:rbac |
| 87gb8fKJHGxh2Pz\_Gk\_R2 | read:users |
| 87gb8fKJHGxh2Pz\_Gk\_R2 | update:devops |
| 87gb8fKJHGxh2Pz\_Gk\_R2 | update:rbac |
| 87gb8fKJHGxh2Pz\_Gk\_R2 | update:users |
| h8Iqlb8Ixc4IltuOoY5QC | create:devops |
| h8Iqlb8Ixc4IltuOoY5QC | delete:devops |
| h8Iqlb8Ixc4IltuOoY5QC | read:devops |
| h8Iqlb8Ixc4IltuOoY5QC | update:devops |
| SbZeBSpuy2OdJ0WZ2Z\_Qo | read:devops |
| SJ36zw7nRS4lx18dZlCoo | create:users |
| SJ36zw7nRS4lx18dZlCoo | delete:users |
| SJ36zw7nRS4lx18dZlCoo | read:users |
| SJ36zw7nRS4lx18dZlCoo | update:users |

### user vs resource permission

> read:R, write:C, delete:D, update:U

| user\_id | devops | rbac | users |
| :--- | :--- | :--- | :--- |
| 87gb8fKJHGxh2Pz\_Gk\_R2 | CDRU | CDRU | CDRU |
| h8Iqlb8Ixc4IltuOoY5QC | CDRU | NULL | NULL |
| SbZeBSpuy2OdJ0WZ2Z\_Qo | R | NULL | NULL |
| SJ36zw7nRS4lx18dZlCoo | NULL | NULL | CDRU |

透過 postgres 的 Transpose Rows to Columns 可以得到上面一個人眼比較直覺得大表
核心的 sql 就是使用 `crosstab` 指令, 完整操作請看下方 full sql

```sql
select *
from crosstab(
             'query order by 1',
             'query distinct resource order by 1'
         ) as (
               user_id text,
               "devops" text,
               "rbac" text,
               "users" text
    )
```

__full sql__

```sql
create table roles
(
    id          varchar(21) not null
        constraint roles_pkey
            primary key,
    parent_id   varchar(21)
        constraint fk_roles_roles
            references roles,
    role_name   text,
    description text,
    type        bigint,
    enabled     boolean,
    created_at  timestamp with time zone,
    updated_at  timestamp with time zone
);


create table permissions
(
    id          varchar(21) not null
        constraint permissions_pkey
            primary key,
    action_id   text,
    resource_id text,
    key         text,
    created_at  timestamp with time zone,
    updated_at  timestamp with time zone
);


create table role_permission
(
    permission_id varchar(21) not null
        constraint fk_role_permission_permission
            references permissions,
    role_id       varchar(21) not null
        constraint fk_role_permission_role
            references roles,
    constraint role_permission_pkey
        primary key (permission_id, role_id)
);


create table actions
(
    id          varchar(21) not null
        constraint actions_pkey
            primary key,
    name        text,
    description text,
    created_at  timestamp with time zone,
    updated_at  timestamp with time zone
);


create table resources
(
    id         varchar(21) not null
        constraint resources_pkey
            primary key,
    name       text,
    created_at timestamp with time zone,
    updated_at timestamp with time zone
);


create table users
(
    id         varchar(21) not null
        constraint users_pkey
            primary key,
    name       text,
    enabled    boolean,
    created_at timestamp with time zone,
    updated_at timestamp with time zone
);


create table user_role
(
    role_id varchar(21) not null
        constraint fk_user_role_role
            references roles,
    user_id varchar(21) not null
        constraint fk_user_role_user
            references users,
    constraint user_role_pkey
        primary key (role_id, user_id)
);


-- data
INSERT INTO actions (id, name, description, created_at, updated_at)
VALUES ('QPjlpxmEE7fSjeTvavn0C', 'create', 'create', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO actions (id, name, description, created_at, updated_at)
VALUES ('2iNamDFktFA6qfz_Tk5nH', 'update', 'update', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO actions (id, name, description, created_at, updated_at)
VALUES ('kZBX-FnIYa374wt4D5OlK', 'delete', 'delete', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO actions (id, name, description, created_at, updated_at)
VALUES ('ozXzHwl1tDLmgIwCBjQMc', 'read', 'read', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');

INSERT INTO resources (id, name, created_at, updated_at)
VALUES ('k4Wcq9zGqShiSH6-HmR9_', 'rbac', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO resources (id, name, created_at, updated_at)
VALUES ('AoiiJEVfm3Fe8gtlZvE0_', 'users', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO resources (id, name, created_at, updated_at)
VALUES ('_-0lB0FJklZQf0wMCOdb3', 'devops', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');

INSERT INTO permissions (id, action_id, resource_id, created_at, updated_at, key)
VALUES ('7BvEdc4TA-gDOF-mfR3Wy', 'ozXzHwl1tDLmgIwCBjQMc', '_-0lB0FJklZQf0wMCOdb3', '2021-01-24T10:24:17+08:00',
        '2021-01-24T10:24:17+08:00', 'read:devops');
INSERT INTO permissions (id, action_id, resource_id, created_at, updated_at, key)
VALUES ('4vIzJkqwRPj2vN_w4Oq88', 'QPjlpxmEE7fSjeTvavn0C', 'k4Wcq9zGqShiSH6-HmR9_', '2021-01-24T10:24:17+08:00',
        '2021-01-24T10:24:17+08:00', 'create:rbac');
INSERT INTO permissions (id, action_id, resource_id, created_at, updated_at, key)
VALUES ('0rhMOepRtB5GPO_6xDLZ3', 'ozXzHwl1tDLmgIwCBjQMc', 'k4Wcq9zGqShiSH6-HmR9_', '2021-01-24T10:24:17+08:00',
        '2021-01-24T10:24:17+08:00', 'read:rbac');
INSERT INTO permissions (id, action_id, resource_id, created_at, updated_at, key)
VALUES ('4P3xauyVkRxinMmKPL6Lh', '2iNamDFktFA6qfz_Tk5nH', 'k4Wcq9zGqShiSH6-HmR9_', '2021-01-24T10:24:17+08:00',
        '2021-01-24T10:24:17+08:00', 'update:rbac');
INSERT INTO permissions (id, action_id, resource_id, created_at, updated_at, key)
VALUES ('S8uK_bjaCDPveyyYqUIJc', 'kZBX-FnIYa374wt4D5OlK', 'k4Wcq9zGqShiSH6-HmR9_', '2021-01-24T10:24:17+08:00',
        '2021-01-24T10:24:17+08:00', 'delete:rbac');
INSERT INTO permissions (id, action_id, resource_id, created_at, updated_at, key)
VALUES ('G_8IyfomSis4Mbk9hqCt9', 'QPjlpxmEE7fSjeTvavn0C', 'AoiiJEVfm3Fe8gtlZvE0_', '2021-01-24T10:24:17+08:00',
        '2021-01-24T10:24:17+08:00', 'create:users');
INSERT INTO permissions (id, action_id, resource_id, created_at, updated_at, key)
VALUES ('6Mh4-cezNheXhIIMmIIjk', 'ozXzHwl1tDLmgIwCBjQMc', 'AoiiJEVfm3Fe8gtlZvE0_', '2021-01-24T10:24:17+08:00',
        '2021-01-24T10:24:17+08:00', 'read:users');
INSERT INTO permissions (id, action_id, resource_id, created_at, updated_at, key)
VALUES ('VgoFSnty6aiBPZUtlsWfg', '2iNamDFktFA6qfz_Tk5nH', 'AoiiJEVfm3Fe8gtlZvE0_', '2021-01-24T10:24:17+08:00',
        '2021-01-24T10:24:17+08:00', 'update:users');
INSERT INTO permissions (id, action_id, resource_id, created_at, updated_at, key)
VALUES ('l4QFZ7gg4K8EhCj_rvduc', 'kZBX-FnIYa374wt4D5OlK', 'AoiiJEVfm3Fe8gtlZvE0_', '2021-01-24T10:24:17+08:00',
        '2021-01-24T10:24:17+08:00', 'delete:users');
INSERT INTO permissions (id, action_id, resource_id, created_at, updated_at, key)
VALUES ('cdemu6I9VAjcYtQURUQJH', 'QPjlpxmEE7fSjeTvavn0C', '_-0lB0FJklZQf0wMCOdb3', '2021-01-24T10:24:17+08:00',
        '2021-01-24T10:24:17+08:00', 'create:devops');
INSERT INTO permissions (id, action_id, resource_id, created_at, updated_at, key)
VALUES ('tIUyyl9n6NKFesDzPlbON', '2iNamDFktFA6qfz_Tk5nH', '_-0lB0FJklZQf0wMCOdb3', '2021-01-24T10:24:17+08:00',
        '2021-01-24T10:24:17+08:00', 'update:devops');
INSERT INTO permissions (id, action_id, resource_id, created_at, updated_at, key)
VALUES ('JK_oVtMSqFPjHklwCedJJ', 'kZBX-FnIYa374wt4D5OlK', '_-0lB0FJklZQf0wMCOdb3', '2021-01-24T10:24:17+08:00',
        '2021-01-24T10:24:17+08:00', 'delete:devops');

INSERT INTO roles (id, parent_id, role_name, type, description, enabled, created_at, updated_at)
VALUES ('AsaN4Lp62jeT9jaJl1Nlj', null, 'admin-manager', 0, 'admin-manager', true, '2021-01-24T10:24:17+08:00',
        '2021-01-24T10:24:17+08:00');
INSERT INTO roles (id, parent_id, role_name, type, description, enabled, created_at, updated_at)
VALUES ('-Nmq8l1U-gfQtgTvxAXDJ', 'AsaN4Lp62jeT9jaJl1Nlj', 'users-manager', 0, 'users-manager', true,
        '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO roles (id, parent_id, role_name, type, description, enabled, created_at, updated_at)
VALUES ('gXWvkQRzMaE6fZnKsbVPD', 'AsaN4Lp62jeT9jaJl1Nlj', 'devops-manager', 0, 'devops-manager', true,
        '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO roles (id, parent_id, role_name, type, description, enabled, created_at, updated_at)
VALUES ('EhZ0CHPt5oc5oe9osk0eQ', 'gXWvkQRzMaE6fZnKsbVPD', 'devops-runner', 0, 'devops-runner', true,
        '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');

INSERT INTO role_permission (role_id, permission_id)
VALUES ('EhZ0CHPt5oc5oe9osk0eQ', '7BvEdc4TA-gDOF-mfR3Wy');
INSERT INTO role_permission (role_id, permission_id)
VALUES ('AsaN4Lp62jeT9jaJl1Nlj', '4vIzJkqwRPj2vN_w4Oq88');
INSERT INTO role_permission (role_id, permission_id)
VALUES ('AsaN4Lp62jeT9jaJl1Nlj', '0rhMOepRtB5GPO_6xDLZ3');
INSERT INTO role_permission (role_id, permission_id)
VALUES ('AsaN4Lp62jeT9jaJl1Nlj', '4P3xauyVkRxinMmKPL6Lh');
INSERT INTO role_permission (role_id, permission_id)
VALUES ('AsaN4Lp62jeT9jaJl1Nlj', 'S8uK_bjaCDPveyyYqUIJc');
INSERT INTO role_permission (role_id, permission_id)
VALUES ('-Nmq8l1U-gfQtgTvxAXDJ', 'G_8IyfomSis4Mbk9hqCt9');
INSERT INTO role_permission (role_id, permission_id)
VALUES ('-Nmq8l1U-gfQtgTvxAXDJ', '6Mh4-cezNheXhIIMmIIjk');
INSERT INTO role_permission (role_id, permission_id)
VALUES ('-Nmq8l1U-gfQtgTvxAXDJ', 'VgoFSnty6aiBPZUtlsWfg');
INSERT INTO role_permission (role_id, permission_id)
VALUES ('-Nmq8l1U-gfQtgTvxAXDJ', 'l4QFZ7gg4K8EhCj_rvduc');
INSERT INTO role_permission (role_id, permission_id)
VALUES ('gXWvkQRzMaE6fZnKsbVPD', 'cdemu6I9VAjcYtQURUQJH');
INSERT INTO role_permission (role_id, permission_id)
VALUES ('gXWvkQRzMaE6fZnKsbVPD', 'tIUyyl9n6NKFesDzPlbON');
INSERT INTO role_permission (role_id, permission_id)
VALUES ('gXWvkQRzMaE6fZnKsbVPD', 'JK_oVtMSqFPjHklwCedJJ');

INSERT INTO users (id, name, enabled, created_at, updated_at)
VALUES ('87gb8fKJHGxh2Pz_Gk_R2', 'User1', true, '2020-09-17 07:56:09.000000', '2021-01-21 15:00:00.000000');
INSERT INTO users (id, name, enabled, created_at, updated_at)
VALUES ('SJ36zw7nRS4lx18dZlCoo', 'User2', true, '2021-01-21 15:00:00.000000', '2021-01-21 15:00:00.000000');
INSERT INTO users (id, name, enabled, created_at, updated_at)
VALUES ('h8Iqlb8Ixc4IltuOoY5QC', 'User3', true, '2021-01-21 15:00:00.000000', '2021-01-21 15:00:00.000000');
INSERT INTO users (id, name, enabled, created_at, updated_at)
VALUES ('SbZeBSpuy2OdJ0WZ2Z_Qo', 'User4', true, '2021-01-21 15:00:00.000000', '2021-01-21 15:00:00.000000');

INSERT INTO user_role (user_id, role_id)
VALUES ('87gb8fKJHGxh2Pz_Gk_R2', 'AsaN4Lp62jeT9jaJl1Nlj');
INSERT INTO user_role (user_id, role_id)
VALUES ('SJ36zw7nRS4lx18dZlCoo', '-Nmq8l1U-gfQtgTvxAXDJ');
INSERT INTO user_role (user_id, role_id)
VALUES ('h8Iqlb8Ixc4IltuOoY5QC', 'gXWvkQRzMaE6fZnKsbVPD');
INSERT INTO user_role (user_id, role_id)
VALUES ('SbZeBSpuy2OdJ0WZ2Z_Qo', 'EhZ0CHPt5oc5oe9osk0eQ');


create or replace view policies_permissions(user_id, key) as
SELECT DISTINCT gpd.user_id,
                p.key
FROM (SELECT gdr.user_id,
             gdr.role_id,
             gdr.role_parent_id,
             rp.permission_id
      FROM (WITH RECURSIVE deep_roles AS (
          SELECT ur.user_id,
                 rr.id,
                 rr.parent_id,
                 1 AS level
          FROM roles rr
                   JOIN user_role ur ON rr.id::text = ur.role_id::text
          WHERE rr.enabled = true
          UNION ALL
          SELECT deep_roles_1.user_id,
                 rr.id,
                 rr.parent_id,
                 deep_roles_1.level + 1 AS level
          FROM roles rr
                   JOIN deep_roles deep_roles_1 ON rr.parent_id::text = deep_roles_1.id::text
          WHERE rr.enabled = true
      )
            SELECT deep_roles.user_id,
                   deep_roles.id        AS role_id,
                   deep_roles.parent_id AS role_parent_id,
                   deep_roles.level
            FROM deep_roles) gdr
               JOIN role_permission rp ON rp.role_id::text = gdr.role_id::text) gpd
         JOIN permissions p ON p.id::text = gpd.permission_id::text;

CREATE EXTENSION IF NOT EXISTS tablefunc;

-- Multi Replace plpgsql - PostgreSQL wiki - https://wiki.postgresql.org/wiki/Multi_Replace_plpgsql
create or replace function quote_meta(text) returns text
    immutable
    strict
    language sql
as
$$
select regexp_replace($1, '([\[\]\\\^\$\.\|\?\*\+\(\)])', '\\\1', 'g');
$$;

--
create or replace function multi_replace(str text, substitutions jsonb) returns text
    immutable
    strict
    language plpgsql
as
$$
DECLARE
    rx     text;
    s_left text;
    s_tail text;
    res    text := '';
BEGIN
    select string_agg(quote_meta(term), '|')
    from jsonb_object_keys(substitutions) as x(term)
    where term <> ''
    into rx;

    if (coalesce(rx, '') = '') then
        -- the loop on the RE can't work with an empty alternation
        return str;
    end if;

    rx := concat('^(.*?)(', rx, ')(.*)$'); -- match no more than 1 row

    loop
        s_tail := str;
        select concat(matches[1], substitutions ->> matches[2]),
               matches[3]
        from regexp_matches(str, rx, 'g') as matches
        into s_left, str;

        exit when s_left is null;
        res := res || s_left;

    end loop;

    res := res || s_tail;
    return res;
END
$$;


select *
from crosstab(
             'select *
                from (select dt.user_id,
                             a[2] as resource,
                             multi_replace(string_agg(a[1], '''' order by a[1]), ''{
                               "delete": "D",
                               "read": "R",
                               "create": "C",
                               "update": "U"
                             }'')  as action
                      from (select regexp_split_to_array(policies_permissions.key, '':''), user_id from policies_permissions) as dt(a)
                      group by 1, 2) as t
                order by 1;',
             'select distinct resource
                from (select dt.user_id,
                             a[2] as resource,
                             multi_replace(string_agg(a[1], '''' order by a[1]), ''{
                               "delete": "D",
                               "read": "R",
                               "create": "C",
                               "update": "U"
                             }'')  as action
                      from (select regexp_split_to_array(policies_permissions.key, '':''), user_id from policies_permissions) as dt(a)
                      group by 1, 2) as t
                order by 1;'
         ) as (
               user_id text,
               "devops" text,
               "rbac" text,
               "users" text
    )


```
