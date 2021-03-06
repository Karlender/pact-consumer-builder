__TODO: Add Maven Central badge here__
[![JCenter](https://api.bintray.com/packages/schuettec/maven/com.remondis.cdc.consumer.pactbuilder/images/download.svg) ](https://bintray.com/schuettec/maven/com.remondis.cdc.consumer.pactbuilder/_latestVersion)
[![Build Status](https://travis-ci.org/remondis-it/pact-consumer-builder.svg?branch=develop)](https://travis-ci.org/remondis-it/pact-consumer-builder)

# Table of Contents
1. [Long Story Short](#long-story-short)
2. [How to use](#how-to-use)
   1. [Custom global data type mappings](#custom-global-data-type-mappings)
   2. [Global Java Bean mappings](#global-java-bean-mappings)
   3. [Declare field mappings](#declare-field-mappings)
3. [How to contribute](#how-to-contribute)

# Long Story Short

This library tries to reduce the overhead of writing the expectations of JSON structures in PACT consumer test. When writing consumer tests, the JSON structures for each endpoint should be defined. The test uses the defined JSON structures as expectations that are matched against the output of a REST endpoint. When the backend uses Java Beans as a representation for JSON structures, this library may help to reduce the overhead of writing the JSON expectations.

The following example shows what is necessary for a PACT consumer test. Assume you want to declare an object holding pricing information, like currency, amount etc. The following PACT consumer test would need the following code:

```
    PactDslJsonBody jsonBody = new PactDslJsonBody().object("currency")
        .numberType("id", expectedPricingResult.getCurrency()
            .getId())
        .stringType("name", expectedPricingResult.getCurrency()
            .getName())
        .stringType("isoCode", expectedPricingResult.getCurrency()
            .getIsoCode())
        .stringType("symbol", expectedPricingResult.getCurrency()
            .getSymbol())
        .closeObject()
        .object("total")
        // and more fields...
``` 

This mapping between the Java Bean and the JSON structure can be generated by this library. All you need to do is to declare what data types are to be mapped. For primitive types a default mapping comes out-of-the-box. For special requirements and custom data types you can specify individual mapping functions on a type or field basis.

The mapping code for the above structure reduces to
```
ConsumerExpects.type(PricingResultResource.class)
    .useTypeMapping(ZonedDateTime.class, (body, fieldName, fieldValue) -> {
      return body.stringType(fieldName, DEFAULT_FORMATTER.format(fieldValue));
    })
    .build(jsonBody, expectedPricingResult);
``` 

The above code shows that all primitive types can be automatically translated to respective calls on `PactDslJsonBody` to declare the JSON structure. The full example uses a field with type `ZonedDateTime`, so a custom converter for this type is added in the example.

You can find the full example [here](src/test/java/com/remondis/cdc/consumer/pactbuilder/testcase/PactFromBeanTest.java)

# How to use

This library converts Java Bean properties in respective calls on the `PactDslJsonBody` API. Here are the data type mappings that are active by default:

| Data type        | Pact DSL Mapping                                                 |
|-----------------|------------------------------------------------------------------|
| String          | pactDslJsonBody.stringValue(fieldName, fieldValue);              |
| byte/Byte       | pactDslJsonBody.numberType(fieldName, fieldValue);               |
| short/Short     | pactDslJsonBody.numberType(fieldName, fieldValue);               |
| int/Integer     | pactDslJsonBody.integerType(fieldName, (Long) fieldValue);       |
| long/Long       | pactDslJsonBody.integerType(fieldName, (Long) fieldValue);       |
| float/Float     | pactDslJsonBody.numberType(fieldName, fieldValue);               |
| double/Double   | pactDslJsonBody.decimalType(fieldName, (Double) fieldValue);     |
| boolean/Boolean | pactDslJsonBody.booleanType(fieldName, fieldValue);              |
| BigDecimal      | pactDslJsonBody.decimalType(fieldName, (BigDecimal) fieldValue); |

## Custom global data type mappings

If you want to map objects that does not comply to the Java Bean convention you can add a custom mapping in the following way:
```
ConsumerExpects.type(SomeJavaBeanType.class)
    .useTypeMapping(NonJavaBeanType.class, (body, fieldName, fieldValue) -> {
      return body.stringType(fieldName, asString(fieldValue));
    })
```
The example shows how to add a mapping for the non-Java Bean type `NonJavaBeanType`. You can specify a function that takes the `PactDslJsonBody`, the field name and an example value for the field and invokes the respective methods on the `PactDslJsonBody` instance. The function must return the resulting `PactDslJsonBody` instance.

Global data type mappings apply to all fields of this type. You can override global mappings with field mappings.

## Global Java Bean mappings

You can reuse Java Bean mappings you declared using this library. If Java Bean references another Java Bean that was already mapped using this library, you can reuse the mapping like this:

```
ConsumerBuilder<Address> addressDefinition = 
    ConsumerExpects.type(Address.class)
    // ... other mapping definitions...
    ;

ConsumerExpects.type(Person.class)
.referencing(addressDefinition)
// ... other mapping definitions...
```
If type `Person` references `Address` and the `Address` structure was already defined, you can reuse the `Address` as a reference from `Person`. When building the JSON structure, the `Address` will be mapped as defined in the registered `ConsumerBuilder`.

## Declare field mappings

 Field mappings are used to override global mappings on a per-field basis or to introduce special cases. You can basically 
 
 * add type mappings for a specific field using a mapping function like described here [Custom global data type mappings](custom-global-data-type-mappings)
 * just define a custom JSON field name while using default mappings
 * reuse another definition for a specific field
 * define a custom JSON field name and reuse another definition for a specific field
 
 Please refer to the JavaDoc or see [here](/src/main/java/com/remondis/cdc/consumer/pactbuilder/FieldBuilder.java)
 

# How to contribute
Please refer to the project's [contribution guide](CONTRIBUTE.md)

