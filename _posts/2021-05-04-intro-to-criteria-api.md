---
title: "Introduction to Criteria API"
categories: [spring-boot]
date: 2021-05-04 05:00:00 +1100
modified: 2021-05-04 05:00:00 +1100
excerpt: "This tutorial is an introduction to Criteria API."
author: hoorvash
image:
  auto: 0016-pen
---

**Criteria API makes it possible to write complex search queries by combining predicates**. It creates query-defining objects to
query our datastore entities in a type-safe and dynamic manner.

{% include github-project.html url="https://github.com/hoorvash/code-examples/tree/criteria-api" %}

## What Is Criteria Api?

**Criteria API offers a programmatic way to create typed queries**. `TypedQuery` is an interface introduced in JPA 2. 
It extends the `Query` interface. `Query` and `TypedQuery` have different usage as mentioned in [ObjectDB](https://www.objectdb.com/java/jpa/query/api) website.

- `Query` interface is mainly used when the query result type is unknown or when a query returns polymorphic results and the lowest known common denominator of all the result objects is `Object`

- `TypedQuery` interface is mainly used when a more specific result type is expected

## When Is Criteria API Useful?
**Benefits of using Criteria API are**:
- **Syntax error decrease**
- **Compile-time-checks when it uses Metalmodel API**
- **Great for complex search queries**
- **Reusable predicates when used by Specifications**

Metamodel API and Specifications are the topics we're going to discuss in this tutorial. But to make a brief explanation: 

- Predicate: **It's part of the criteria package and is used to construct the `where` clause**
- Metamodel API: **It provides the ability to retrieve details on persistent object models, 
so it's kind of what reflection is towards general java types**
- Specification: **It's what makes Criteria API predicates reusable and eliminates boilerplates**

Let's define our entity object `Product` first:

````java
@Entity
@Table(name = "t_product")
public class Product {

  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  private Long id;

  @Column(name = "c_name")
  private String name;

  @Column(name = "c_category")
  private String category;

  @Column(name = "c_price")
  private String price;

  //  getter and setter

}
````

## Create Query with Criteria Api
We're going to retrieve products that have specific names and categories using Criteria API.

````java
@Repository
public class ProductRepositoryImpl {
        
  @PersistenceContext
  EntityManager em;

  public List<Product> findProductsByNameAndCategory(
                                            String name,
                                            String category) {
      
  CriteriaBuilder cBuilder = em.getCriteriaBuilder();
  CriteriaQuery<Product> cQuery = cBuilder.createQuery(Product.class);

  Root<Product> root = cQuery.from(Product.class);

  /* Conditions **/
  Predicate namePredicate = cBuilder.
    like(root.get("name"), '%' + (name == null ? "" : name) + '%');
  Predicate categoryPredicate = cBuilder.
    equal(root.get("category"), (category == null ? Category.TOOLS : category));

  cQuery.where(namePredicate, categoryPredicate);

  /* Create Query object **/
  TypedQuery<Product> query = em.createQuery(cQuery);

  return query.getResultList();
  }
}
````

The above code's description is as follows:

* First two lines of the method shows how to get `CriteriaBuilder` and `CriteriaQuery` to create different parts of the query
* Variable `root` is like `FROM table_name` in our query declaration introduced to the `cQuery` object
* Variables `namePredicate` and `categoryPredicate` are defined as conditions in the query. Only applied if used
as parameters of `where` method of `cQuery` object.
* We finally create the `TypedQuery` object with `em.createQuery(cQuery)`. `TypedQuery` interface is a
 subclass of `Query` interface.
 
 Objects `namePredicate` and `categoryPredicate` are predicates that construct `where` clause.

## JPA Metamodel
In the above code example, We used the name of the fields of the entity classes in the predicates like `"name"` and `"category"`.
**Using string names of entities in the predicates has some flaws**:
- **It's hard to remember the name so we might need to look it up**
- **Query will need refactoring if the column's name change later**
- **It increases the chance of errors in runtime because of the possible mistype** 

**Using JPA Metamodels fixes the above problems**. **It helps us to avoid using the column's name**. It does
 that by generating some classes similar to the name of corresponding entities but with an "_" at the end**.
In this tutorial, we're going to use the Metamodel generator tool provided by [JBoss](https://docs.jboss.org/hibernate/orm/5.0/topical/html/metamodelgen/MetamodelGenerator.html).

The first step is to add Metamodel Generator dependency and also a required plugin in `pom.xml` as compiler argument: 

````xml
  <dependencies>
      <!-- Other required dependencies ... -->
      <dependency>
          <groupId>org.hibernate</groupId>
          <artifactId>hibernate-jpamodelgen</artifactId>
          <version>5.3.7.Final</version>
      </dependency>
  </dependencies>
    
````

````xml   
  <build>
      <plugins>
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
          </plugin>
          <plugin>
              <artifactId>maven-compiler-plugin</artifactId>
              <configuration>
                  <source>11</source>
                  <target>11</target>
                  <compilerArguments>
                      <processor>org.hibernate.jpamodelgen.JPAMetaModelEntityProcessor</processor>
                  </compilerArguments>
              </configuration>
          </plugin>
      </plugins>
  </build>
```` 
By building the project, the Metamodel classes will be generated automatically in the `target/generated-classes` folder. 
**We can access all our entity object fields through the generated classes. Metamodel class name for `Product` entity is `Product_` 
like the following**: 

````java
@Generated(value = "org.hibernate.jpamodelgen.JPAMetaModelEntityProcessor")
@StaticMetamodel(Product.class)
public abstract class Product_ {

  public static volatile SingularAttribute<Product, Double> price;
  public static volatile SingularAttribute<Product, String> name;
  public static volatile SingularAttribute<Product, Long> id;
  public static volatile SingularAttribute<Product, String> category;

  public static final String PRICE = "price";
  public static final String NAME = "name";
  public static final String ID = "id";
  public static final String CATEGORY = "category";

}
````

We repeat `findProductsByNameAndCategory` implementation, this time we're using Metamodel classes in the `Predicates` 
definition as you can see here:

````java
@Repository
public class ProductRepositoryImpl {

  @PersistenceContext
  EntityManager em;

  public List<Product> findProductsByNameAndCategory(
                                      String name,
                                      String category) {

    CriteriaBuilder cBuilder = em.getCriteriaBuilder();
    CriteriaQuery<Product> cQuery = cBuilder.createQuery(Product.class);

    /* FROM **/
    Root<Product> root = cQuery.from(Product.class);

    /* Conditions using Metamodel **/
    Predicate namePredicate = cBuilder.
      like(root.get(Product_.NAME), '%' + (name == null ? "" : name) + '%');
    Predicate categoryPredicate = cBuilder.
      equal(root.get(Product_.CATEGORY), (category == null ? Category.TOOLS : category));

    /* WHERE **/
    cQuery.where(namePredicate, categoryPredicate);

    /* Create Query object **/
    TypedQuery<Product> query = em.createQuery(cQuery);

    return query.getResultList();
  }
}
````
**By using Metamodel we can use `Product_.name` and `Product_.category` instead of `"name"` and `"category"`**.

## Specifications
**Specifications build on top of the Criteria API** to simplify the developer experience. Let's take a look at the Spring 
Data Jpa query method's downsides first. Then we see how it can be better by JPA Specification. 

````java
interface ProductRepository {
  List<Product> findAllByNameLikeAndPriceLessThanEqual(
          String name,
          Double price
  );
}
````
**Using the Spring Data query methods has some problems**:

* **The length of the query method might increase significantly depending on number of criteria**
* **The number of query methods increases rapidly as the use cases increases**
* **There are many overlapping criteria across the query methods**
* **We can only specify a fixed number of criteria**
* **If there is a change in any one of these criteria variables, we’ll have to make changes in multiple query methods**

**By using Specifications we can build atomic predicates, and combine those predicates to build complex dynamic queries**.
In the following code, we can see a sample of using Specifications to create some atomic predicates:

````java
public class ProductSpecification {

  public static Specification<Product> belongsToCategory(
    List<Category> categories) {
      return (root, query, cBuilder) ->
        cBuilder.
        in(root.get(Product_.CATEGORY)).value(categories);
  }

  public static Specification<Product> hasNameLike(String name) {
    return ((root, cQuery, cBuilder) -> 
      cBuilder.
      like(root.get(Product_.NAME), '%' + (name == null ? "" : name) + '%'));
  }

  public static Specification<Product> isCheap() {
    return ((root, cQuery, cBuilder) ->
      cBuilder.
      lessThan(root.get(Product_.PRICE), 10));
  }
}
````

To execute these Specifications, `ProductRepository` should `extends` an interface named `JpaSpecificationExecutor`.

````java
@Repository
public interface ProductRepository extends
     JpaRepository<Product, Long> ,
     JpaSpecificationExecutor<Product> {

  // query methods here
  List<Product> findByName(String name);
  // query methods here
}
````

Now we see this Specification execution:

````java
  productRepository.findAll(ProductSpecification.isCheap());
````

### Combine Specifications
Combining the predicates happens via `Specification` interface `public static` helper methods `and()`, `or()` and
`where()` to get the desired result. 

````java
  public static Specification<Product> getCheapProductsWithNameLike(String name) {
      return Specification.where(ProductSpecification.isCheap().and(
              ProductSpecification.hasNameLike(name)
      ));
  }
````
As we can see, the structure of the method still does not support dynamic queries. **To achieve a dynamic
search method, generate the predicates more general via the Specifications**. To dynamically combine multiple criteria
Let's take a look at the code here:

````java
public enum Operation {
  EQ, GT, LT, LIKE;
  public static final String[] SIMPLE_OPERATION_SET =
          { ":", ">", "<", "%" };
  public static Operation getSimpleOperation(final char input)
  {
      switch (input) {
          case ':': return EQ;
          case '>': return GT;
          case '<': return LT;
          case '%': return LIKE;
          default: return null;
      }
  }
}

````
````java
public class Criteria {
  private String key;
  private Object value;
  private Operation operation;
  private boolean orPredicate;
}
````
We create a `Criteria` class that contains the `Operation` field.

Our purpose is to get a `String` input. Then parse it to get the desired query. After that, we pass it through 
`ProductSpecificationBuilder` to generate the proper Specification and retrieve the data. 
The input String of our dynamic search method is going to be like this: `name:broom,price<30`. 
This means that we want a product that its name is `equal` to `broom` and its `price` is less than `30$`.

The `Criteria` implementation holds a basic representation of a constraint meaning:
- key: The field that the query is based on it - for example : `price`, `name`, ...etc
- value: The value of the field - for example : 10, pizza, ... etc
- operation: It's the condition - for example :  equality, like, ...etc
- orPredicate: It shows `or` or `and` between specifications 

Let's see a simple `ProductSpecificationBuilder` implementation:

````java
public class ProductSpecificationBuilder {

  private final List<Criteria> params;

  public ProductSpecificationBuilder() {
      params = new ArrayList<>();
  }

  public final ProductSpecificationBuilder with(
        final String key, final String operation, final Object value) {
      return with(false, key, operation, value);
  }

  public final ProductSpecificationBuilder with(
                            final boolean orPredicate,
                            final String key,
                            final String operation,
                            final Object value) {

      Operation op = Operation.getSimpleOperation(operation.charAt(0));
      if (op != null) {
          params.add(new Criteria(orPredicate, key, op, value));
      }
      return this;
  }

  public Specification<Product> build() {
      
      if (params.size() == 0)
        return null;

      Specification<Product> result = new ProductSpecification(params.get(0));

      for (int i = 1; i < params.size(); i++) {
          result = params.get(i).isOrPredicate()
                  ? Objects.requireNonNull(Specification.where(result)).
                      or(new ProductSpecification(params.get(i)))
                  : Objects.requireNonNull(Specification.where(result)).
                      and(new ProductSpecification(params.get(i)));
      }
      return result;
  }
}
````
````java
public class ProductSpecification implements Specification<Product> {

  private Criteria criteria;

  public ProductSpecification(final Criteria criteria) {
      super();
      this.criteria = criteria;
  }

  public Criteria getCriteria() {
      return criteria;
  }

  @Override
  public Predicate toPredicate(
     final Root<Product> root, final CriteriaQuery<?> cQuery, final CriteriaBuilder cBuilder) {

       switch (criteria.getOperation()) {
           case EQ:
             return cBuilder.equal(
                root.get(criteria.getKey()), criteria.getValue());
           case GT:
             return cBuilder.greaterThan(
                root.get(criteria.getKey()), criteria.getValue().toString());
           case LT:
             return cBuilder.lessThan(
                root.get(criteria.getKey()), criteria.getValue().toString());
           case LIKE:
             return cBuilder.like(
                root.get(criteria.getKey()), criteria.getValue().toString());
           default:
             return null;
       }
  }
}
````

In `ProductSpecification`, we execute the Specifications.

The `params` in `ProductSpecificationBuilder` is the separated SpecificationCriteria that is extracted from input String.
So if the input is  `name:broom,price<30` with a false value in `orPredict` field, the params size is `2`.
- First is: `Criteria={key:"name", value:"broom", operation:"EQ", orPredicate: false}`
- Second is: `Criteria={key:"price", value:"30", operation:"LT", orPredicate: false}`
And there's a `and` between these two specifications because of `orPredicate` value.


Let's implement a Rest API, and use the Specifications. It's better to write some part of the code to a service layer, 
but for simplicity, we write all the code in the controller. 

````java
@RestController
@RequestMapping("/api")
public class ProductController {
    
  private final ProductRepository productRepository;

  public ProductController(ProductRepository productRepository) {
      this.productRepository = productRepository;
  }

  @GetMapping(value = "/product/{search}/{predicate}")
  public List<Product> search(
                @PathVariable(value = "search") String search,
                @PathVariable(value = "predicate") PredicateEnum predicate) {

      ProductSpecificationBuilder specificationBuilder = new ProductSpecificationBuilder();
      Pattern pattern = Pattern.compile("(\\w+?)([:<%>])(\\w+?),");
      Matcher matcher = pattern.matcher(search + ",");
      while (matcher.find()) {
          if (predicate.equals(PredicateEnum.OR)) {
              specificationBuilder.with(true, matcher.group(1), matcher.group(2), matcher.group(3));
          } else {
              specificationBuilder.with(matcher.group(1), matcher.group(2), matcher.group(3));
          }
      }
      Specification<Product> spec = specificationBuilder.build();
      return productRepository.findAll(spec);
  }
}
````  
The full implementation of this tutorial is on [Github](https://github.com/hoorvash/code-examples/tree/criteria-api).
We can access the Rest API with URL `http://localhost:1112/swagger-ui.html#/` to test this query: `name:broom,price<30`.
We can learn more about Specification from [Getting Started with Spring Data Specifications](https://reflectoring.io/spring-data-specifications/).

## Conclusion
Criteria API with the use of the JPA Specification creates a powerful dynamic search query.
This tutorial discusses how to implement Criteria API, first without any concern on boilerplate codes. Then
It offers a simple implementation of a dynamic search API. Of course, it can be
extended to a more complex need.
