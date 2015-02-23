
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

The first step leading to UI pattern DSL will be to provide automatic assemblers. So users won't have to write them, even if it will be still configurable.

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

Assemble.dto(customerDto).to(Customer.class).from().repository("remote").or().repository("local").or().factory();

Assemble.dto(customerDto).to(Customer.class).from().repository().withAssembler(new AbstractMapping() {
        public void configureMerge(MyDTO source) {
            skip(source.name);
        }
});

Assemble.aggregate(customer).to(CustomerDTO.class).withMapping(new AbstractMapping() {
        public void configureAssembly(MyAggregate source) {
            skip(source.name);
        }
});

Assemble.aggregate(customer).toUniversalDto();

Assemble.dto(universalDto).to(Customer.class);
```    

## Automatic finders

```


```


Two separate improvements should be done to finders:

* Simplify the existing paginated finder to 


## Resource templates

...

## UI pattern DSL

...

## Hypermedia

...

 