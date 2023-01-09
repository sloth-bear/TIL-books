Principles of Object Oriented Design

0. Dont' Repeat Yourself (DRY)
- Duplication in logic calls for abstraction
- Duplication in process calls for automation

1. Single Responsibility
* single reason to change
* Dependency and coupling 
  - 서로 다른 concept을 커플링하지 않기 

* Common Refactoring 
  - Extract class  
  - Move Method
  

2. Open/Closed
* Open to extension 
  - New behavior can be added in the future

* Closed to Modification 
  - Changes to source or binary code ar not required

* Common Refactoring 
  - Extract Interace / Apply Strategy Pattern 
  - Parameterize Method 
  - Form Template Method 

3. Liskov Substitution
* Sounds complicated 

* You might be breaking this principle 
    - Call a method on derived type and get NotImplementedException
    - Get a base type but still have to check what is the actual type

* Comman Refactoring 
  - Collapse Hierarchy
  - Pull up / Push Down Field 
  - Pull Up / Push Down Method 

4. Interface Segregation
* Common Refactoring 
  - Extract Interface 
  - Extract Adapter

5. Dependency Inversion
* Depend on Abstractions 
  - interfaces, not concrete types 

* Inject Dependencies into Classes

* Structure Solution so Dependencies Flow Toward Core
  - Onion Architecture



============================

Where do we start?

* seperation of Concerns: Single Responsiblity 
  >>>> Not only for the prod code but also TEST CODE!
* Don't repeat yourself
* Extract Helper Classes
* Define Dependencies


