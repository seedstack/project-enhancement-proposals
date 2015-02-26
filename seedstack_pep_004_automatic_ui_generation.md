
- PEP: 004  
- Title: Automatic UI generation (codenamed Kaviar)  
- Authors: 
 - Adrien LAUER <adrien.lauer@mpsa.com>
 - Kavi RAMYEAD <kavi.ramyead@ext.mpsa.com>
 - Pierre THIROUIN <pierre.thirouin@ext.mpsa.com>
 - Redouane LOULOU <redouane.loulou@ext.mpsa.com>  
- Status: draft  

# Abstract

Implements features needed to automatically expose representations. We already have automatic repository and factory. We now need:
- Automatic assembler
- Automatic finder
- Resource templates
- Hypermedia links between representations
- Standard UI patterns

# Motivation

We want the business framework to be a complete productivity framework based on DDD principles. The creation of DDD building blocks has already been addressed, so in this PEP we focus on speeding up development with the business framework.

# Specification


## Automatic Assembler

This part is located in business framework core.

The first step leading to automatic UI will be to provide automatic assemblers. So users won't have to write them, even if it will be still configurable.

In order to do this, we will create a specific assembler extending `AbstractBaseAssembler`. It will use the DSL provided by [ModelMapper](http://modelmapper.org/).

This draft needs some complements on:

* POC with maps and lists
* POC with tuples
* Data security

```java
public class AutomaticAssembler<AGGREGATE_ROOT extends AggregateRoot<?>, DTO extends Object> extends AbstractBaseAssembler<AGGREGATE_ROOT, DTO>{
...
}
```

`AutomaticAssembler` will keep the compatibility with the current assemblers by specializing them for automatic mapping. The do...() methods will already be implemented in those automatic assemblers with the ability for end users to override a configure() method if special configuration is needed. An explicitly written assembler class will not be needed if the developer want to keep the default behavior (much like the default factories or repositories). In that case, an annotation must be applied on the DTO class for the framework to recognize the association to the aggregate. **This part will supplement the currently existing assemblers.**

```java
public MyAssembler extends AutomaticAssembler<MyAggregateRoot, MyDTO> {

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

On top of that an API and/or a DSL would be provided for the developer to specify the whole assembling behavior, including loading from a repo, creating from a factory and so on... **This part would completely replace the current Assemblers facade, which should be deprecated and reimplemented with this DSL**.

```java
Assemble.dto(customerDto).to(customer);

Assemble.dto(customerDto).to(Customer.class).fromRepositories("qualifier1", "qualifier2").thenFromFactories("qualifier3", "qualifier4");

Assemble.dto(customerDto).to(Customer.class).fromRepository().thenFromFactory();

Assemble.dto(customerDto).to(Customer.class).fromRepository().withAssembler(new AbstractMapping() {
        public void configureMerge(MyDTO source) {
            skip(source.name);
        }
});

Assemble.aggregate(customer).to(CustomerDTO.class).withMapping(new AbstractMapping() {
        public void configureAssembly(MyAggregate source) {
            skip(source.name);
        }
});

Assemble.aggregate(customer).toDynamicDto();

Assemble.dto(dynamicDto).to(Customer.class);
```    

## Finder improvements

This part is located in business framework core.

Advanced finders often requires multiple features like pagination, filtering or sorting. One should be able to compose such features for each specific finder. For instance, a finder could implement pagination and sorting but not filtering. An API based on a decorator pattern could provide this flexibility instead of inheritance (inability to compose) or interface composition (inability to implement default behavior).

A default finder implementation should be provided (like default factories, repositories, ...) which will have a simple but effective implementation of at least the three following features: pagination, filtering, sorting.

A DSL will enable to use the finders (whether a default or an explicit implementation) at a higher level of abstraction. Finder features will be easily addressable:

```
Find.chunk(15, 35).of(CustomerDTO.class).withFinder("qualifier1")
    .sortedBy("firstName").asc()
    .sortedBy("lastName")
    .filteredBy(theCriteria);

Find.page(5, 10).of(CustomerDTO.class)
    .sortedBy("firstName").asc()
    .sortedBy("lastName")
    .filteredBy(theCriteria);

Find.all().of(CustomerDTO.class);
```

## Resource templates

This part is located in business framework rest.

The idea is to allow the creation of resource templates which will be like normal resources but not automatically registered. Instead they can be exposed multiple times with different representations and various configurations.

```
Expose.representation(CustomerRepresentation.class).with(CRUDResourceTemplate.class).on("/customers");

Expose.dynamicRepresentationOf(Customer.class).with(CRUDResourceTemplate.class).on("/customers");

Expose.representation(CustomerRepresentation.class).with(CRUDResourceTemplate.class).on("/customers")
    .usingFactory("qualifier1")
    .usingRepository("qualifier2")
    .usingFinder("qualifier3")
    .usingAssembler("qualifier4");
```

## Hypermedia

This part is located in business framework rest.

## UI patterns

This part is located in the kaviar function.
