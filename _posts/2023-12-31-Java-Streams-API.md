---
layout: post
title: Java Streams API
subtitle: Improving Readability, Focussing on What instead of How!
# gh-repo: daattali/beautiful-jekyll
# gh-badge: [star, fork, follow]
thumbnail-img: /assets/img/streamApi.jpg
tags: [Java, Functional Programming]
comments: true
author: Pulkit Mahajan
---

The Java Stream API simplifies the task of data processing, reduces verbosity, and enhances code expressiveness to a much greater extent. Streams represent sequences of elements and support operations like filtering, sorting, mapping, and collecting. By chaining these operations into a pipeline, one can query data efficiently. Streams are lazy, parallelizable, and align well with functional programming concepts, making them a powerful addition to the Collections API.

### The normal Way!
Lets say we have a service that provides a method that returns all the People in the database. Now our task is to print the first 10 persons whose age is less than 18. 
```java
List<People> people = service.getPeople();
int count = 0; 
for(People p: people){
    if(p.getAge()<=18){
        System.out.println(p);
        count++;  
    }
    if(count==10)break;
} 
```

### The Mentos Way!
```java
List<People> people = service.getPeople(); 
people.stream()
    .filter(person -> person.getAge()<=18)
    .limit(10)
    .collect(Collectors.toList())
    .forEach(System.out::println); 
```

I am assuming that you already know the core stuff of how everything in java streams api works and all. This page shall be more of a summary page, that lists down all the functionalities of java streams API along with appropriate code snippets to get you going with your task at hand. 


**Iterating** over a range. This is similar to what people are used to with  
`for(int i = 0 ; i < n ; i++)`.  
```java
    //example 1
    IntStream.range(0,5).forEach(System.out::println); 

    //example 2
    List<Person> persons = service.getPeople(); 
    IntStream.range(0,persons.size())       // this part has already created a stream!
        //.limit(5)                         // if you wish to limit the output. 
        //.filter(number -> number%2)       // if you only wish odd numbers
        .forEach(index -> {
            Person person = persons.get(index); 
            System.out.println(person); 
    });
```

Suppose you just queried a list of all user_ids that logged in to your application on a particular day, and now you wish to find all the **unique** users. 
```java
List<Integer> userIds = service.fetchAllUserIds(); 
Set<Integer> distinctNums = userIds.stream()
            .collect(Collectors.toSet()); 
```
You Should always try out looking at all the Collectors that you have at hand, before you start thinking of complex logics.  

Suppose you have a list of all your Persons, and you wish to find the average age of all people. The similar method can be used to find the sum of all ages as well. 
```java
Double avg = persons.stream()
        .mapToDouble(person::getAge) 
        .average().orElse(0);          // because average() would return an Optional Object. 
System.out.println(avg); 
```
Another handy method that comes to use is `.summaryStatistics()`. This provides you the *average, Count, Max, Min, Sum* for the list you got. 

<!-- Find Any?? -->

##### GroupBy 
I found this one really interesting! Its similar to the groupby Function of pandas if you ever used that. 
You can even try to create nested groups if you wish to. 
```java
Map<String, List<Person>> groupingByGender = service.getPeople().stream()
                .filter(person -> person.getAge()>=18) 
                .collect(Collectors.groupingBy(Person::getGender)); 

// grouping at a more deeper level. 
HashMap<String, Map<String, List<Person>>> groupingByCity = new HashMap<>(); 
groupingByGender.forEach((gender,listOfPersons) -> {
    Map<String, List<Person>> subGrouping = listOfPersons.stream()
            .collect(Collectors.groupingBy(Person::getCity)); 
    groupingByCity.put(gender,subGrouping)
})
```
Again Collectors to the rescue! This function is quite handly and never to be underestimated. 

There is another use case where you can group and count as well. I could not thoroughly understand that, but here you go. 
```java
Map<String, Long> collect = names.stream().collect(
    Collectors.groupingBy(Function.identity(), Collectors.counting())); 
collect.forEach((name, count)->{
    System.out.println(name + " " + count.toString()); 
}); 
```

##### Reduce
```java
// Still working on this. no info found on the course
```

Then there's another thing called the FlatMap. In a nutshell, you'll use it when you have a list that is nested at multiple levels, and you are only concerned with every as an individual in a list that is *not* nested. This can also be done with the reduce method, but that would be a little bit more pain.
```java
List<String> newList = nestedList.stream()
                .flatMap(List::stream).collect(Collectors.toList()); 
```

Remember the convenient `join` operation in `Python`. Well... see for yourself. Again, you could have done this easily with reduce as well.  
```java
System.out.println(names.stream().collect(Collectors.joining(", "))); 
```

### Parallel Streams 
This is where stuff gets really goood. 
`working on this`







