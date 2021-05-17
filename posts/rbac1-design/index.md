# Rbac1 Design


<!--more-->

權限系統在任何應用程式都是基礎且重要的部份，主要是對不同的人訪問資源進行權限的控制，本篇文章並沒有特別的再說明 Role-Based Access Control, 解釋的文章已經太多了，隨便 Google 都是一堆。最期在公司的專案也是有採用 RBAC 來管控系統的權限，一般的 RBAC0 (最基本的 User - Role - Permission) 的模型大至上可以符合需求，不過在權限配直的時候會有一點重工，所以選擇了有角色繼承的 RBAC1 來進行實作

{{<image src="/posts/rbac1-design/img/rbac1.jpeg" caption="RBAC1">}}

{{<image src="/posts/rbac1-design/img/public.png" caption="Postgres Database Schema">}}

### public.actions

column | comment | type | length | default | constraints | values
--- | --- | --- | --- | --- | --- | ---
**id** _(pk)_ |  | character varying | 21 |  | NOT NULL | 
name |  | text |  |  |  | 
description |  | text |  |  |  | 
created_at |  | timestamp with time zone |  |  |  | 
updated_at |  | timestamp with time zone |  |  |  | 

### public.permissions

column | comment | type | length | default | constraints | values
--- | --- | --- | --- | --- | --- | ---
**id** _(pk)_ |  | character varying | 21 |  | NOT NULL | 
action_id |  | text |  |  |  | 
resource_id |  | text |  |  |  | 
key |  | text |  |  |  | 
created_at |  | timestamp with time zone |  |  |  | 
updated_at |  | timestamp with time zone |  |  |  | 

### public.resources

column | comment | type | length | default | constraints | values
--- | --- | --- | --- | --- | --- | ---
**id** _(pk)_ |  | character varying | 21 |  | NOT NULL | 
name |  | text |  |  |  | 
created_at |  | timestamp with time zone |  |  |  | 
updated_at |  | timestamp with time zone |  |  |  | 

### public.role_permission

column | comment | type | length | default | constraints | values
--- | --- | --- | --- | --- | --- | ---
**permission_id** _(pk)_ |  | character varying | 21 |  | NOT NULL, [permission_id](#public-permissions) | 
**role_id** _(pk)_ |  | character varying | 21 |  | NOT NULL, [role_id](#public-roles) | 

### public.roles

column | comment | type | length | default | constraints | values
--- | --- | --- | --- | --- | --- | ---
**id** _(pk)_ |  | character varying | 21 |  | NOT NULL | 
parent_id |  | character varying | 21 |  | [parent_id](#public-roles) | 
role_name |  | text |  |  |  | 
description |  | text |  |  |  | 
type |  | bigint |  |  |  | 
enabled |  | boolean |  |  |  | 
created_at |  | timestamp with time zone |  |  |  | 
updated_at |  | timestamp with time zone |  |  |  | 

### public.user_role

column | comment | type | length | default | constraints | values
--- | --- | --- | --- | --- | --- | ---
**role_id** _(pk)_ |  | character varying | 21 |  | NOT NULL, [role_id](#public-roles) | 
**user_id** _(pk)_ |  | character varying | 21 |  | NOT NULL, [user_id](#public-users) | 

### public.users

column | comment | type | length | default | constraints | values
--- | --- | --- | --- | --- | --- | ---
**id** _(pk)_ |  | character varying | 21 |  | NOT NULL | 
name |  | text |  |  |  | 
enabled |  | boolean |  |  |  | 
created_at |  | timestamp with time zone |  |  |  | 
updated_at |  | timestamp with time zone |  |  |  | 


```sql
create table roles
(
	id varchar(21) not null
		constraint roles_pkey
			primary key,
	parent_id varchar(21)
		constraint fk_roles_roles
			references roles,
	role_name text,
	description text,
	type bigint,
	enabled boolean,
	created_at timestamp with time zone,
	updated_at timestamp with time zone
);


create table permissions
(
	id varchar(21) not null
		constraint permissions_pkey
			primary key,
	action_id text,
	resource_id text,
	key text,
	created_at timestamp with time zone,
	updated_at timestamp with time zone
);


create table role_permission
(
	permission_id varchar(21) not null
		constraint fk_role_permission_permission
			references permissions,
	role_id varchar(21) not null
		constraint fk_role_permission_role
			references roles,
	constraint role_permission_pkey
		primary key (permission_id, role_id)
);


create table actions
(
	id varchar(21) not null
		constraint actions_pkey
			primary key,
	name text,
	description text,
	created_at timestamp with time zone,
	updated_at timestamp with time zone
);


create table resources
(
	id varchar(21) not null
		constraint resources_pkey
			primary key,
	name text,
	created_at timestamp with time zone,
	updated_at timestamp with time zone
);


create table users
(
	id varchar(21) not null
		constraint users_pkey
			primary key,
	name text,	
	enabled boolean,
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
INSERT INTO public.actions (id, name, description, created_at, updated_at) VALUES ('QPjlpxmEE7fSjeTvavn0C', 'create','create', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO public.actions (id, name, description, created_at, updated_at) VALUES ('2iNamDFktFA6qfz_Tk5nH', 'update','update', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO public.actions (id, name, description, created_at, updated_at) VALUES ('kZBX-FnIYa374wt4D5OlK', 'delete','delete', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO public.actions (id, name, description, created_at, updated_at) VALUES ('ozXzHwl1tDLmgIwCBjQMc', 'read','read', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');

INSERT INTO public.resources (id, name, created_at, updated_at) VALUES ('k4Wcq9zGqShiSH6-HmR9_', 'rbac', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO public.resources (id, name, created_at, updated_at) VALUES ('AoiiJEVfm3Fe8gtlZvE0_', 'users', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO public.resources (id, name, created_at, updated_at) VALUES ('_-0lB0FJklZQf0wMCOdb3', 'devops', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');

INSERT INTO public.permissions (id, action_id, resource_id, created_at, updated_at, key) VALUES ('7BvEdc4TA-gDOF-mfR3Wy', 'ozXzHwl1tDLmgIwCBjQMc', '_-0lB0FJklZQf0wMCOdb3', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00', 'read:devops');
INSERT INTO public.permissions (id, action_id, resource_id, created_at, updated_at, key) VALUES ('4vIzJkqwRPj2vN_w4Oq88', 'QPjlpxmEE7fSjeTvavn0C', 'k4Wcq9zGqShiSH6-HmR9_', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00', 'create:rbac');
INSERT INTO public.permissions (id, action_id, resource_id, created_at, updated_at, key) VALUES ('0rhMOepRtB5GPO_6xDLZ3', 'ozXzHwl1tDLmgIwCBjQMc', 'k4Wcq9zGqShiSH6-HmR9_', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00', 'read:rbac');
INSERT INTO public.permissions (id, action_id, resource_id, created_at, updated_at, key) VALUES ('4P3xauyVkRxinMmKPL6Lh', '2iNamDFktFA6qfz_Tk5nH', 'k4Wcq9zGqShiSH6-HmR9_', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00', 'update:rbac');
INSERT INTO public.permissions (id, action_id, resource_id, created_at, updated_at, key) VALUES ('S8uK_bjaCDPveyyYqUIJc', 'kZBX-FnIYa374wt4D5OlK', 'k4Wcq9zGqShiSH6-HmR9_', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00', 'delete:rbac');
INSERT INTO public.permissions (id, action_id, resource_id, created_at, updated_at, key) VALUES ('G_8IyfomSis4Mbk9hqCt9', 'QPjlpxmEE7fSjeTvavn0C', 'AoiiJEVfm3Fe8gtlZvE0_', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00', 'create:users');
INSERT INTO public.permissions (id, action_id, resource_id, created_at, updated_at, key) VALUES ('6Mh4-cezNheXhIIMmIIjk', 'ozXzHwl1tDLmgIwCBjQMc', 'AoiiJEVfm3Fe8gtlZvE0_', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00', 'read:users');
INSERT INTO public.permissions (id, action_id, resource_id, created_at, updated_at, key) VALUES ('VgoFSnty6aiBPZUtlsWfg', '2iNamDFktFA6qfz_Tk5nH', 'AoiiJEVfm3Fe8gtlZvE0_', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00', 'update:users');
INSERT INTO public.permissions (id, action_id, resource_id, created_at, updated_at, key) VALUES ('l4QFZ7gg4K8EhCj_rvduc', 'kZBX-FnIYa374wt4D5OlK', 'AoiiJEVfm3Fe8gtlZvE0_', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00', 'delete:users');
INSERT INTO public.permissions (id, action_id, resource_id, created_at, updated_at, key) VALUES ('cdemu6I9VAjcYtQURUQJH', 'QPjlpxmEE7fSjeTvavn0C', '_-0lB0FJklZQf0wMCOdb3', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00', 'create:devops');
INSERT INTO public.permissions (id, action_id, resource_id, created_at, updated_at, key) VALUES ('tIUyyl9n6NKFesDzPlbON', '2iNamDFktFA6qfz_Tk5nH', '_-0lB0FJklZQf0wMCOdb3', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00', 'update:devops');
INSERT INTO public.permissions (id, action_id, resource_id, created_at, updated_at, key) VALUES ('JK_oVtMSqFPjHklwCedJJ', 'kZBX-FnIYa374wt4D5OlK', '_-0lB0FJklZQf0wMCOdb3', '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00', 'delete:devops');

INSERT INTO public.roles (id, parent_id, role_name, type, description, enabled,  created_at, updated_at) VALUES ('AsaN4Lp62jeT9jaJl1Nlj', null, 'admin-manager', 0, 'admin-manager', true, '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO public.roles (id, parent_id, role_name, type, description, enabled,  created_at, updated_at) VALUES ('-Nmq8l1U-gfQtgTvxAXDJ', 'AsaN4Lp62jeT9jaJl1Nlj', 'users-manager', 0, 'users-manager', true, '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO public.roles (id, parent_id, role_name, type, description, enabled,  created_at, updated_at) VALUES ('gXWvkQRzMaE6fZnKsbVPD', 'AsaN4Lp62jeT9jaJl1Nlj', 'devops-manager', 0, 'devops-manager', true, '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');
INSERT INTO public.roles (id, parent_id, role_name, type, description, enabled,  created_at, updated_at) VALUES ('EhZ0CHPt5oc5oe9osk0eQ', 'gXWvkQRzMaE6fZnKsbVPD', 'devops-runner', 0, 'devops-runner', true, '2021-01-24T10:24:17+08:00', '2021-01-24T10:24:17+08:00');

INSERT INTO public.role_permission (role_id, permission_id) VALUES ('EhZ0CHPt5oc5oe9osk0eQ', '7BvEdc4TA-gDOF-mfR3Wy');
INSERT INTO public.role_permission (role_id, permission_id) VALUES ('AsaN4Lp62jeT9jaJl1Nlj', '4vIzJkqwRPj2vN_w4Oq88');
INSERT INTO public.role_permission (role_id, permission_id) VALUES ('AsaN4Lp62jeT9jaJl1Nlj', '0rhMOepRtB5GPO_6xDLZ3');
INSERT INTO public.role_permission (role_id, permission_id) VALUES ('AsaN4Lp62jeT9jaJl1Nlj', '4P3xauyVkRxinMmKPL6Lh');
INSERT INTO public.role_permission (role_id, permission_id) VALUES ('AsaN4Lp62jeT9jaJl1Nlj', 'S8uK_bjaCDPveyyYqUIJc');
INSERT INTO public.role_permission (role_id, permission_id) VALUES ('-Nmq8l1U-gfQtgTvxAXDJ', 'G_8IyfomSis4Mbk9hqCt9');
INSERT INTO public.role_permission (role_id, permission_id) VALUES ('-Nmq8l1U-gfQtgTvxAXDJ', '6Mh4-cezNheXhIIMmIIjk');
INSERT INTO public.role_permission (role_id, permission_id) VALUES ('-Nmq8l1U-gfQtgTvxAXDJ', 'VgoFSnty6aiBPZUtlsWfg');
INSERT INTO public.role_permission (role_id, permission_id) VALUES ('-Nmq8l1U-gfQtgTvxAXDJ', 'l4QFZ7gg4K8EhCj_rvduc');
INSERT INTO public.role_permission (role_id, permission_id) VALUES ('gXWvkQRzMaE6fZnKsbVPD', 'cdemu6I9VAjcYtQURUQJH');
INSERT INTO public.role_permission (role_id, permission_id) VALUES ('gXWvkQRzMaE6fZnKsbVPD', 'tIUyyl9n6NKFesDzPlbON');
INSERT INTO public.role_permission (role_id, permission_id) VALUES ('gXWvkQRzMaE6fZnKsbVPD', 'JK_oVtMSqFPjHklwCedJJ');

INSERT INTO public.users (id, name, enabled, created_at, updated_at) VALUES ('87gb8fKJHGxh2Pz_Gk_R2', 'User1', true, '2020-09-17 07:56:09.000000', '2021-01-21 15:00:00.000000');
INSERT INTO public.users (id, name, enabled, created_at, updated_at) VALUES ('SJ36zw7nRS4lx18dZlCoo', 'User2', true, '2021-01-21 15:00:00.000000', '2021-01-21 15:00:00.000000');
INSERT INTO public.users (id, name, enabled, created_at, updated_at) VALUES ('h8Iqlb8Ixc4IltuOoY5QC', 'User3', true, '2021-01-21 15:00:00.000000', '2021-01-21 15:00:00.000000');
INSERT INTO public.users (id, name, enabled, created_at, updated_at) VALUES ('SbZeBSpuy2OdJ0WZ2Z_Qo', 'User4', true, '2021-01-21 15:00:00.000000', '2021-01-21 15:00:00.000000');

INSERT INTO public.user_role (user_id, role_id) VALUES ('87gb8fKJHGxh2Pz_Gk_R2','AsaN4Lp62jeT9jaJl1Nlj');
INSERT INTO public.user_role (user_id, role_id) VALUES ('SJ36zw7nRS4lx18dZlCoo','-Nmq8l1U-gfQtgTvxAXDJ');
INSERT INTO public.user_role (user_id, role_id) VALUES ('h8Iqlb8Ixc4IltuOoY5QC','gXWvkQRzMaE6fZnKsbVPD');
INSERT INTO public.user_role (user_id, role_id) VALUES ('SbZeBSpuy2OdJ0WZ2Z_Qo','EhZ0CHPt5oc5oe9osk0eQ');


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
```

在繼承的關係下，`admin-manager` 會有以下角色 (`user-manager`, `devops-manager`, `devops-runner`) 所有的權限。那我們怎麼取得 `user1`, `user3`, `user3`, `user4` 在角色繼承下的權限呢，這時候我們就可以透過 CTE (common table expression) 來詢查，我們進行小的查詢，最後組成一個 view

1. ```sql
     SELECT ur.user_id,
                 rr.id,
                 rr.parent_id,
                 1 AS level
          FROM roles rr
                   JOIN user_role ur ON rr.id::text = ur.role_id::text
    ```

    | user\_id | id | parent\_id | level |
    | :--- | :--- | :--- | :--- |
    | 87gb8fKJHGxh2Pz\_Gk\_R2 | AsaN4Lp62jeT9jaJl1Nlj | NULL | 1 |
    | SJ36zw7nRS4lx18dZlCoo | -Nmq8l1U-gfQtgTvxAXDJ | AsaN4Lp62jeT9jaJl1Nlj | 1 |
    | h8Iqlb8Ixc4IltuOoY5QC | gXWvkQRzMaE6fZnKsbVPD | AsaN4Lp62jeT9jaJl1Nlj | 1 |
    | SbZeBSpuy2OdJ0WZ2Z\_Qo | EhZ0CHPt5oc5oe9osk0eQ | gXWvkQRzMaE6fZnKsbVPD | 1 |
    
    查詢 user 對應到 role 預先代入 level 的欄位給 recursive 查詢使用
2. ```sql
    WITH RECURSIVE deep_roles AS (
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
      ) select * from deep_roles
    ```

    | user\_id | id | parent\_id | level |
    | :--- | :--- | :--- | :--- |
    | 87gb8fKJHGxh2Pz\_Gk\_R2 | AsaN4Lp62jeT9jaJl1Nlj | NULL | 1 |
    | SJ36zw7nRS4lx18dZlCoo | -Nmq8l1U-gfQtgTvxAXDJ | AsaN4Lp62jeT9jaJl1Nlj | 1 |
    | h8Iqlb8Ixc4IltuOoY5QC | gXWvkQRzMaE6fZnKsbVPD | AsaN4Lp62jeT9jaJl1Nlj | 1 |
    | SbZeBSpuy2OdJ0WZ2Z\_Qo | EhZ0CHPt5oc5oe9osk0eQ | gXWvkQRzMaE6fZnKsbVPD | 1 |
    | 87gb8fKJHGxh2Pz\_Gk\_R2 | gXWvkQRzMaE6fZnKsbVPD | AsaN4Lp62jeT9jaJl1Nlj | 2 |
    | 87gb8fKJHGxh2Pz\_Gk\_R2 | -Nmq8l1U-gfQtgTvxAXDJ | AsaN4Lp62jeT9jaJl1Nlj | 2 |
    | h8Iqlb8Ixc4IltuOoY5QC | EhZ0CHPt5oc5oe9osk0eQ | gXWvkQRzMaE6fZnKsbVPD | 2 |
    | 87gb8fKJHGxh2Pz\_Gk\_R2 | EhZ0CHPt5oc5oe9osk0eQ | gXWvkQRzMaE6fZnKsbVPD | 3 |

    使用 recursive 詢查之後，我們會得到 user 對到的 role, 並有 level 欄位可以看到繼承的深度

3. ```sql
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
    ```

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

    最後一部份就是將經過 recursive 查詢後的 user - role 表去 join permission 拿到 user 對應 role 之後的權限表，然後將這一個查詢作成 View 方便後序使用

4. Demo RBAC1
      {{<image src="/posts/rbac1-design/img/rbac1.png" caption="Demo RBAC1">}}


### DB fiddle Playground
https://www.db-fiddle.com/f/4jyoMCicNSZpjMt4jFYoz5/1179
