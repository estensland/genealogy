# Genealogy

## Premise
Genealogy is still under development and need to be improved and extended. Developed features so far were the ones I needed for my personal applications but also the basic ones for a genealogy managment system that's why I decided to extract it in a gem. So please use with care but, above all, use it with critical and contructive sense.


## Description

Genealogy is a ruby gem library which extend ActiveRecord::Base class with familiar relationships capabilities in order to build and query genealogies. If records of your model need to be linked and act as they were individuals of a big family just add two parents column to its table (ie.: *father_id* and *mother_id*) and make your model to *:has_parents*. This macro will provide your model with the two fundamental self-join associations, *father* and *mother*, whose everything depend on.  
Genealogy takes inspiration from the simple [linkage file format](http://www.helsinki.fi/~tsjuntun/autogscan/pedigreefile.html) which represent genealogies in terms of set of trios: *individual-id*, *father-id*, *mother-id*. Basically the only **primitive** familiar relationships are associations father and mother, all others like grandparents, siblings or offspring are **derived**. This means that all methods in charge of update the genealogy (adding/removing relatives) will end up to use the fundamental method `add/remove_parent` to the right records

## Installation

To apply Genealogy in its simplest form to any ActiveRecord model, follow these simple steps:  

1. Install   
    1. Add to Gemfile: gem ‘genealogy’   
    2. Install required gems: bundle install    

2. Add the foreign key parents columns to your table     
    1. Create migration: `rails g migration add_parents_to_[table] father_id:integer mother_id:integer`. A **sex column is also required**, add it if not exists.
    2. Add index separately and in combination to parents columns   
    3. Migrate your database: `rake db:migrate`

3. Add `has_parents` to your model

## Usage

Instance methods are two kinds: *queries* and *modifiers*

### Query methods

These self explaining methods simply parse the tree through parents' associations to answer query like:

* `george.father` will retrieve george's father
* `george.parents` will retrieve george's parents as 2 elements sorted array: [father,mother] 
* `george.paternal_grandfather` 
* `george.paternal_grandparents` will return a 2 elements sorted array: [paternal_grandfather, paternal_granmother]
* `george.grandparents` will return a 4 elements sorted array: [paternal_grandfather, paternal_granmdother, maternal_grandfather, maternal_grandmother]

* `george.ancestors` will return an unsorted array
* `george.descendants` idem

* `george.siblings` will return full-siblings array (same father and mother)
* `george.siblings(:half => :mother)` will return maternal half-siblings array (same mother)
* `george.siblings(:half => :only)` will return only half-siblings 
* `george.siblings(:half => :include)` will return only full-siblings and half-siblings all toghether

* `george.offspring` will return all individuals that have george as father (mother can be any)
* `george.offspring(:spouse => :gina)` will return all individuals that have george as father and gina as mother


### Modifiers methods

They change the genealogy updating one ore more target individuals (the ones that are actually modified). These methods overwrite old values and call internally *save!* so all validation and callbacks are regularly run. Here are some examples where target individual correspond with method's receiver:

* `george.add_father(peter)` will change george's father_id to peter.id
* `george.add_father(charlie)` will overwrite george's father_id
* `george.add_mother(gina)` 
* `george.add_parents(peter,gina)` 

Not always the method's receiver is the instance that get updated (target record):

* `george.add_paternal_grandmother(gina)` will change mother of george's father. No changes for george and gina. It can seem weird but think about it as an intuitive shortcut to quickly build the genealogy as a whole entity.

* `george.add_paternal_grandparents(julius,marta)` will change parents of george's father like the *add_parents* method does.

Some methods can take a list of individuals:

* `george.add_grandparents(paul,gina,nick,mary)` will change both george's parents
* `george.add_grandparents(paul,nil,nick,mary)` you can skip unknown relatives using nil

* `julian.add_siblings(gina,rita)` will turn gina and rita's parents into the julian's ones

Multiple mating are supported so half parent relationships are admitted and can build separately:

* `luke.add_offspring(julian)` will set only one of julian's parent depending on luke's gender. By the way, given luke is a male, this is of course equivalent to `julian.add_father(luke)`

* `luke.add_offspring(julian, :spouse => gina)` will also change julian's mother

* `walter.add_siblings(luke, :half => :father)` will add luke as a paternal half-sibling, that is only luke's father will be changed to walter's one

* `walter.add_siblings(luke, :half => :father, :spouse => magda)` will also change luke's mother to magda. This let set a different mother for the sibling

Methods that involves updates of two or more instances use transactions to ensure the good outcome of the whole operation:

* `george.add_grandparents(paul,gina,nick,mary)` when george's mother is invalid will raise an exception and no one of george's parents get updated.

Removing methods examples are:

* `george.remove_father` will set to nil george's father_id 
* `george.remove_parents` will set both parents to nil
* `george.remove_paternal_grandfather` will set father george's father to nil
* `george.remove_paternal_grandparents` will set both george's parent to nil
* `george.remove_grandparents` will set all parents' parents to nil
*
 `george.remove_offspring` will nullify the father of all that records that have george as father
* `george.remove_offspring(:affect_spouse => true)` will also nullify mother of all that records that have george as father
* `george.remove_offspring(:spouse => gina)` will nullify the father of all that records that have george as father and gina as mother
* `george.remove_offspring(:spouse => gina, :affect_spouse => true)` will do both last two actions

* `george.remove_siblings` will nullify both full-siblings parents
* `george.remove_siblings(:half => :father)` will nullify only father of all records that have same george's father as father
* `george.remove_siblings(:half => :father, :affect_spouse => true)` will nullify also mother of all records that have same george's father as father

### *has_parents* options

pending

### Test as documentation

A rich amount of test examples were written using the RSpec suite. Beyond the canonical testing purpouse I tried to make the test output as readable as possible in order to serve as auxiliary documentation. Just type *rake* to run all of them and get the output in a pleasant human readable format.



