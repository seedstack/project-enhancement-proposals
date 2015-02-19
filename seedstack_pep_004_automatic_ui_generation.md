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
public class DefaultAssembler {

    @Inject
    Map<String, Mapper> mappers

    public Object assembleDtoFromAggregate(Object sourceAggregate,
    Class<?> dtoClass) {
        Mapper mapper = mappers.get(dtoClass.getSimpleName())
        ModelMapper modelMapper = mapper.getModelMapper();
        PropertyMap mappings = mapper.getAssembleMappings();
        if (mappings != null) {
            modelMapper.addMappings(mappings())
        }
        return modelMapper.map(sourceAggregate, dtoClass);
    }

    public void updateDtoFromAggregate(Object sourceDto, Object sourceAggregate) {
        ModelMapper modelMapper = getModelMapper();
        modelMapper.addMappings(mappers.get(...).getUpdateMappings())
        return modelMapper.map(sourceAggregate, getDtoClass());
    }

    public void mergeAggregateWithDto(Object targetAggregate, Object sourceDto) {
        ModelMapper modelMapper = getModelMapper();
        modelMapper.addMappings(mappers.get(...).getMergeMappings())
        return modelMapper.map(sourceDto, getAggregateClass());
    }

}
```

```java
public interface Mapper<AGGREGATE, DTO> {

    ModelMapper getModelMapper();

    PropertyMap<AGGREGATE, DTO> getAssembleMappings();

    PropertyMap<AGGREGATE, DTO> getUpdateMappings();

    PropertyMap<DTO,AGGREGATE> getMergeMappings();
}
```

```java
public class BaseMapper implements Mapper<AGGREGATE, DTO> {

    ...
}
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

 
