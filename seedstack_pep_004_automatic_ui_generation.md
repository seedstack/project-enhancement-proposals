
PEP: 004  
Title: Automatic UI generation (codenamed Kaviar)  
Authors: Adrien LAUER <adrien.lauer@mpsa.com>, Kavi RAMYEAD <kavi.ramyead@ext.mpsa.com>, 
Pierre Thirouin <pierre.thirouin@ext.mpsa.com>, Redouane LOULOU <redouane.loulou@ext.mpsa.com>  
Status: draft  

# Abstract

...

# Motivation

...

# Specification


## Automatic Assembler

The first step leading to UI pattern DSL will be to provide automatic
assemblers. So users won't have to write them, even if it will be
still configurable.

In order to do this, we will create a specific assembler implementing
`AbstractBaseAssembler`. It will use the DSL provided by
[ModelMapper](http://modelmapper.org/).

This draft needs some complements on:
* POC with maps and lists
* POC with tuples
* Data security ?

```java
public class DefaultAssembler<AGGREGATE_ROOT extends AggregateRoot<?>, DTO extends Object> extends AbstractBaseAssembler<AGGREGATE_ROOT, DTO>{
...
}
```

`DefaultAssembler` will keep the compatibility with the current assemblers by specializing them for automatic mapping. The do...() methods would already be implemented in those automatic assemblers with the ability for end users to override a configure() method if special configuration is needed. An explicitly written assembler class would not be needed if the developer want to keep the default behavior (much like the default factories or repositories). **This part would supplement the currently existing assemblers.**

```java
public MyAssembler extends DefaultAssembler<MyDTO, MyAggregateRoot> {

    public void configureAssembly(MyAggregateRoot source) {
        map().getAddress().setStreet(source.getStreet());
        skip(source.name);
    }

    public void configureMerge(MyDTO source) {
        map().getAddress().setStreet(source.getStreet());
    }

}
```
Note that map(), skip() and other methods should detect from which method it is called to provide the right target.

An interface, implemented by AutomaticAssembler, could specify the mapping configuration behavior (configureMerge and configureAssembly) to be reused in the DSL.

On top of that an API and/or a DSL would be provided for the developer to specify the whole assembling behavior, including loading from a repo, creating from a factory and so on... **This part would completely replace the current Assemblers facade, which should be deprecated and reimplemented with this DSL**.

```java
Map.dto(customerDto).to(customer);

Map.dto(customerDto).to(Customer.class).from().repository("remote").or().repository("local").or().factory();

Map.dto(customerDto).to(Customer.class).loadedFromRepository().withMapping(new AbstractMapping() {
        public void configureMerge(MyDTO source) {
            skip(source.name);
        }
});

Map.aggregate(customer).to(CustomerDTO.class).withMapping(new AbstractMapping() {
        public void configureAssembly(MyAggregate source) {
            skip(source.name);
        }
});

Map.aggregate(customer).toUniversalDto();

Map.dto(universalDto).to(Customer.class);
```    

## Automatic finders

Pagination with criteria.

Looks for BaseSimpleJpaFinder.

## Resource templates

...

## UI pattern DSL

...

## Hypermedia

...

 