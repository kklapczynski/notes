### Strings
- for string comparisions use only .equals or .equalsIgnoreCase; "==" checks memory location and separate strings are always separate locations in memory
    - "Hello".substring(0,3) == "Hel"  => returns false
- .length() returns number of code units ( not Unicode characters - some like emotikons are build from 2 code units)
- there is no method that mutates string - they always return a new one

### Numbers
- int and long for normal use
- BigInteger or BigDecimal objects for bigger numbers

### Structures
- "for each" loop:
```JAVA
    int[] a = new int[100];

    for (int element : a) {}
```

### Arrays
- array initialazers
```JAVA
int[] sP = {2,3,5,7,11,13};
```
- normal assignment copies reference not an array itself
```JAVA
int[] luckyNumbers = sP;    // changing value in one array chnages in the other as well
```
- to do true copy of array:
```JAVA
int[] copiedLuckyNumbers = Arrays.copyOf(luckyNumbers, luckyNumbers.length);
```

### Classes
## Fields
- __private__ - set fields of class as private - access only with class methods
- __final__ - field cannot change = cannot be replaced by other object, but still it can be mutated
    ```JAVA
        private final StringBuilder evaluations;
        public Employee() { evaluations = new StringBuilder(); ...}
        public void giveGoldStar() { evaluations.append("Gold star!\n)"; }
    ```
- __static__ 
    - __static__ FIELD - field of a class not an instance; shared across all instances
        - e.g. to keep track of instances id:
            ```JAVA
                private static int nextId;  // 1 field per CLASS
                private int id; // 1 field per INSTANCE
                public void setId() { id = nextId; nextId++ }   // sets id of instance, changes class id for next instance
            ```
        - to keep shared constants
            ```JAVA
                public class Math {
                    public static final double PI = 3.14;   // accessible anywhere as Math.PI
                }
            ```
    - __static__ METHOD
        - doesn't operate on objects like Math.pow(a,b)
        - doesn't use "this"
        - to do something just with parameters of a method with no need for any object
        - static method cannot access instance field, only static field = class static fields
### Parameters
- in JAVA they are CALLED BY VALUE : copy of parameter is created and metod works on it only = cannot change the parameter passed into method
    ```JAVA
        public static void tripleValue(double x) { x = 3 * x;}
        double percent = 10;
        tripleValue(percent);   // after this percent is still 10
    ```
- above is opposite to OBJECTS passed as parameters = method can mutate OBJECT passed as param
    - a variable holding objects is actually a reference to an object
    - method works on a copy of param = copy of reference
    - both references are anyway to the same object
    - after method returns its param copy disapears, but object to which it was referncing persist
### Constructors
- a class can have more then 1 constructor - they differ on parameters - constructor name is __OVERLOADED__
- __method signature__ = name + parameters types
- a field that isn't set explicitly in constructor gets a default value depending on its type: all numbers -> 0, booleans -> false, objects -> null
- if a class has no constructor then it sets all field to default
- if a class has at least 1 constructor, the 'no-argument' constructor is not provided
- __constructor parametes names__ - for better documentation use full words
    - with prefix 'a'
        ```JAVA
            public Employee(String aName, double aSalary) {
                name = aName;
                salary = aSalary;
            }
        ```
    - with 'this'
        ```JAVA
            public Employee(String name, double salary) {
                this.name = name;
                this.salary = salary;
            }
        ```
- constructor can call another constructor in the __first statement__
    ```JAVA
        public Employee(double s) {
            // calls another constructor Employee(String, double)
            this("Employee #" + nextID, s);
            nextId++;
        }
    ```
## CLASS DESIGN HINTS
- always keep data private - all instance fields declared as private
- always initialize instance fields
- don't use too many basic types (strings and numbers), many of them is a hint that other classes should be created
- don't add getter and setter for each field, just for the ones really needed
- break up classes when they have to many methods - group and create new classes
- all names should describe object they represent
- prefer immutable classes

### Inheritance
- __extend__ a class: ```JAVA public class Manager extends Employee```
- __super__ (https://docs.oracle.com/javase/tutorial/java/IandI/super.html); (https://learning.oreilly.com/videos/core-java-11/9780135160053/9780135160053-CJ92_01_05_03)
    - use in a subclass definition to call superclass methods
        ```JAVA
            public class Superclass {

                public void printMethod() {
                    System.out.println("Printed in Superclass.");
                }
            }
            // Here is a subclass, called Subclass, that overrides printMethod()

            public class Subclass extends Superclass {

                // overrides printMethod in Superclass
                public void printMethod() {
                    super.printMethod();    // calls method from superclass
                    System.out.println("Printed in Subclass");
                }
                public static void main(String[] args) {
                    Subclass s = new Subclass();
                    s.printMethod();    
                }
            }
        ```
    - use superclass constructor to add own initialization code; Invocation of a superclass constructor must be the first line in the subclass constructor.
        ```JAVA
           public MountainBike(int startHeight, 
                    int startCadence,
                    int startSpeed,
                    int startGear) {
                super(startCadence, startSpeed, startGear);
                seatHeight = startHeight;
            }       
        ```
- block extending a class with __final__
    ```JAVA public final class Executive extends Manager { }```
- block method overriding with __final__
- __abstract__  - to declare method without implementation - it will be provided in subclasses
```JAVA public abstract String getDescription();```
- class with abstract methods must be declared as well as abstract
```JAVA public abstract class Person```
- abstract class can have normal fields, constructors and methods, but cannot be instantiated
(https://learning.oreilly.com/videos/core-java-11/9780135160053/9780135160053-CJ92_01_05_03)
- __protected__ field - accessible for subclasses, any method from any subclass can access it, but only in subclass instances (instead of calling ```JAVA super.methodInSuperClass()```); danger is that protected field cannot be removed, cause some subclasses using it might stop to work. Generally not good to use with fields, but sometimes __protected__ methods are useful

### Object class and its methods
- https://learning.oreilly.com/videos/core-java-11/9780135160053/9780135160053-CJ92_01_05_04
- https://github.com/kklapczynski/JAVA_fundamentals_examples/tree/master/corejava/v1ch05/equals
- created classes might need to have:
    - a way to compare its instances - need to overwrite __equals__ method
    - a way to produce hashCode of them - need to overwrite __hashCode__ method (overwrite _hashCode_ method whenever you overwrite _equals_, if objects are equal their hashcodes must be equal as well): if equals method was comparing 3 fields: name, salary, hireDay, then those need to be in hashcode method:
    ```JAVA
        public class Employee {
            ...
            public int hashCode() {
                return Objects.hash(name, salary, hireDay);
            }
        }
    ```
    - a way to return string representation of an object - use __toString__ method (used for debugging for example)
    ```JAVA
        public String toString() {
            return getClass().getName()
                + "[name=" + name + ", salary-" + salary + "]";
        }
    ```


- everything apart from primitives is an instance of object class
- __Objects.equals(field, otherField)__ is null safe
- to compare class use __getClass()__ ```JAVA if(getClass() != otherObject.getClass()) return false;```
- to compare instances use __instanceOf__ ```JAVA if(!(otherObject instanceOf Person)) return false;```

### ArrayList
- https://learning.oreilly.com/videos/core-java-11/9780135160053/9780135160053-CJ92_01_05_05
- used a lot cause Array length is fixed
- is array of objects that can grow and shrink
- it's generic: type parameter specifies element type
```JAVA ArrayList<Employee> staff = new ArrayList<>();```
- __add()__ method to add object at the end ```JAVA staff.add(new Employee(".."));```
- __size()__ returns current length
- __get__ and __set__ ```JAVA Employee e = staff.get(i); staff.set(i, tony);```
- __"for each"__ loop ```JAVA for (Employee e : staff) System.out.println(e);```
- cannot hold int values, but __Integer__ object wraps int and conversion is automatic
```JAVA
    ArrayList<Integer> list = new ArrayList<>();
    list.add(3);
    int n = list.get(i);
    Integer n = 1000;
    n++;    // as well works cause of this auto conversion = "boxing + unboxing" ("wrapping + unwrapping")
    Integer a = n + 1;
    Integer b = n + 1;
    return a == b;  // returns false cause a and b are differetn objects
    // need to use .equals
    // int cannot be null, but Integer can, so below error may occur
    Integer n = null;
    System.out.println(n+1); // Null pointer exeption
```
### Methods with variable number of parameters
- use "..." in method definition - puts separate args into array that can be itarated  with for loop
```JAVA
    public static double max(double... values) {
        double largets = Double.NEGATIVE_INFINITY;
        for(double v : values) if (v > largest) largest = v;
        return largest;
    }

    double d = max(1,2,3,44,5,7) // returns 44
```
### Enumeration classes
- it defines all instances
```JAVA
    public enum Size { SMALL, MEDIUM, LARGE, EXTRA_LARGE };
```
- methods:
    - __toString__ - returns string "SMALL", ...
    - __ordinal__  - returns position 0, ...
    - static methods:
        - __Enum.valueOf(Size.class, "SMALL")__ - returns Size.SMALL
        - Size.values() yields all values in an array of type Size[]



