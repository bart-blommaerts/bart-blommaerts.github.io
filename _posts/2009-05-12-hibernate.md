---
layout: post
authors: [bart_blommaerts]
title: "HibernateException: Unexpected row count"
tags: [hibernate]
category: Java
comments: true
---

For little more than a month I've been coaching a Junior Java Developer on a project for the <a title="Flemish Government" href="http://www.vlaanderen.be" target="_blank">Flemish Government</a>. A couple of days ago, he came to me with a problem with his persistance layer.  The application uses an <a title="Oracle" href="http://www.oracle.com" target="_blank">Oracle</a> 10g database and uses <a title="Hibernate" href="https://www.hibernate.org/" target="_blank">Hibernate</a> for Object Relational Mapping. Transactions and DI are configured with <a title="Spring" href="http://www.springsource.org/" target="_blank">Spring</a>. The problem itself might seem trivial, but finding the solution has cost me over a day and a half ..

The relevant part of the datamodel:

<p style="text-align: center;">  
  <img class="image fit" alt="schoolchildparent" src="/img/older/schoolchildparent.png">
</p>

1 school may have several children.
1 child may have several parents. <em>(preferrably)</em>

Relevant DDL:
<pre>create table SCHOOL (
   ID number(19,0) not null enable,
   NAME varchar2(200),
   constraint SCHOOL_PK primary key (ID)
);

create table CHILD (
   ID number(19,0) not null enable,
   SCHOOL_ID number(19,0) not null enable,
   NAME varchar2(200),
   constraint CHILD_PK primary key (ID),
   constraint CHILD_SCHOOL_FK foreign key (SCHOOL_ID) references SCHOOL(ID) enable
);

CREATE SEQUENCE CHILD_SEQ
   INCREMENT BY 1 START WITH 1 MAXVALUE 9.999999999999999e+26
   MINVALUE 1 NOCYCLE CACHE 20 NOORDER;

create or replace trigger TR_CHILD
before insert on CHILD for each row
begin
   select CHILD_SEQ.NEXTVAL
   INTO :NEW.id
   from DUAL;
end;

create table PARENT (
   ID number(19,0) not null enable,
   CHILD_ID number(19,0) not null enable,
   NAME varchar2(200),
   constraint PARENT_PK primary key (ID),
   constraint PARENT_CHILD_FK foreign key (CHILD_ID) references CHILD(ID) enable
);

CREATE SEQUENCE PARENT_SEQ
   INCREMENT BY 1 START WITH 1 MAXVALUE 9.999999999999999e+26
   MINVALUE 1 NOCYCLE CACHE 20 NOORDER;

create or replace trigger TR_PARENT
before insert on PARENT
for each row
begin
   select PARENT_SEQ.NEXTVAL
   INTO :NEW.id
   from DUAL;
end;</pre>
Nothing strange going on here: creating tables with constraints, sequences and triggers for autogenerating an id, when inserting a new record.

I'm not going into creating Pojo's and DAO's as that is not part of the problem or the solution.

Hibernate mappings looked like this. Classes are mapped bi-directional:
<pre>    &lt;class name="be.vlaanderen.School" table="SCHOOL"&gt;
        &lt;id name="id" column="ID"&gt;
            &lt;generator class="sequence"&gt;
                &lt;param name="sequence"&gt;SCHOOL_SEQ&lt;/param&gt;
            &lt;/generator&gt;
        &lt;/id&gt;
        &lt;property name="name" column="NAME" /&gt;

        &lt;set name="children" cascade="all-delete-orphan"&gt;
            &lt;key column="SCHOOL_ID"/&gt;
            &lt;one-to-many class="be.vlaanderen.Child"/&gt;
        &lt;/set&gt;
    &lt;/class&gt;

    &lt;class name="be.vlaanderen.Child" table="CHILD"&gt;
        &lt;id name="id" column="ID"&gt;
            &lt;generator class="sequence"&gt;
                &lt;param name="sequence"&gt;CHILD_SEQ&lt;/param&gt;
            &lt;/generator&gt;
        &lt;/id&gt;
        &lt;many-to-one name="school" column="SCHOOL_ID" /&gt;
        &lt;property name="name" column="NAME" /&gt;

        &lt;set name="parents" cascade="all-delete-orphan"&gt;
            &lt;key column="PARENT_ID"/&gt;
            &lt;one-to-many class="be.vlaanderen.Parent"/&gt;
        &lt;/set&gt;
    &lt;/class&gt;

    &lt;class name="be.vlaanderen.Parent" table="PARENT"&gt;
        &lt;id name="id" column="ID"&gt;
            &lt;generator class="sequence"&gt;
                &lt;param name="sequence"&gt;PARENT_SEQ&lt;/param&gt;
            &lt;/generator&gt;
        &lt;/id&gt;
        &lt;many-to-one name="child" column="CHILD_ID" /&gt;
        &lt;property name="name" column="NAME" /&gt;
    &lt;/class&gt;</pre>
At first sight, nothing strange or incorrent. However when his DAO called the
<pre>getHibernateTemplate().saveOrUpdate(school);</pre>
he always received the following error:
<pre>HibernateException: Unexpected row count: 0 expected: 1</pre>
This error means that somewhere along the way something happened behind Hibernate's back that changed the id of the entities we are trying to persist, because of the cascade mode. In code however, no id's were manually changed at all. It appeared that Hibernate was fighting with itself on how to update the id of the child-records. After a while, we changed the Set in School to:
<pre>        &lt;set name="children" cascade="all-delete-orphan" inverse="true"&gt;
            &lt;key column="SCHOOL_ID"/&gt;
            &lt;one-to-many class="be.vlaanderen.Child"/&gt;
        &lt;/set&gt;</pre>
This was a (partial) succes. The child records were persisted and correctly linked to the school-record. When we applied the same change to the parent set, we didn't have the same success: the parent-records had no reference anymore to the child-records. Changing <a title="inverse" href="http://simoes.org/docs/hibernate-2.1/155.html" target="_blank">inverse</a> from false (default) to true, changed the responsibilty of the relationship. Thus: changes in Child weren't persisted in the database, so if something changed the id of child, we didn't notice it in the database. When persisting parent, we had no refernce to child, because the id <strong>did</strong> change. <em>(I dislike the name inverse, something like "owner" would be easier to understand)</em>

In the end, the problem wasn't with Hibernate .. but with the database. We created a Trigger in Oracle "BEFORE INSERT". Due this trigger, Hibernate and Oracle were fighting on who should set the foreign keys. Nothing was wrong with the initial mapping and after dropping the triggers, the application worked as expected. The HibernateException was correct and understandable (for once): something changed the id. Whilst most developers would start searching in the code (as I did), it's important to remember that triggers and sequences in a DBMS are also responsible for changing and linking id's.