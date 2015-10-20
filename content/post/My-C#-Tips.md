---
author: Hetal Patel
comments: true
date: 2015-06-12 05:20:03+00:00
layout: post
slug: My Tips
title: My C# Tips
wordpress_id: 22
categories:
- nothing
---

### The Null Coalescing operator

The null-coalescing operator can simplify our If statements when we need to check for the existence of a null value and provide some other default. If we don't use the null-coalescing operator, we typically end up with If statements that look something like this. 

            if (name == null)
            {
                result = "no name provided";
            }
            else
            {
                result = name;
            }

Now Let using the null-coalescing operator. 

            var name = "Hetal";

            var result = name ?? "no name provided";

Here, we set a name and then we want to get a result. What we want is if the name is not null then to just use the name. Otherwise, use this default string. The ?? here signify the null-coalescing operator.



### Testing char Unicode validity

It's useful to check whether a character is valid or not. To do this, we can use the static GetUnicodeCharacter method of the char struct. To use this method, we pass in a char and we get back a category. To test whether the character is valid, we check this Category is not equal to the OtherNotAssigned category.

            var validCharacter = 'q';

            var ucCategory = char.GetUnicodeCategory(validCharacter);

            var isValidUnicode = ucCategory != UnicodeCategory.OtherNotAssigned;  

isValidUnicode is true because at the moment we're comparing the lowercase letter q.

If we cast from an integer and convert this to a char, we could end up with something that is not a valid Unicode character.
 
            var invalidCharacter = (char) 888;

            ucCategory = char.GetUnicodeCategory(invalidCharacter);

            isValidUnicode = ucCategory != UnicodeCategory.OtherNotAssigned;
            
  The category that we've got is the OtherNotAssigned category and isValidUnicode is now false. So the GetUnicodeCategory method is a useful one whenever we're dealing with Unicode conversions that may fall outside of the valid set of Unicode values. Unicode categories include other useful things. We can find out whether the categories are currency symbols, enclosing marks, letter number separators, punctation, and a category such as uppercase letters.

### Creating random numbers

When we create a new random instance without specifying a seed value, it uses the current system time as the seed value. Because the system clock is limited in its level granularity, if we create two random instances close together, they may be initialized with the same seed value and hence will produce the same sequence of numbers. in the example below I've created two random instances, r1 and r2, and then I simply generated 5 numbers from that random instance. I've then done the same thing again, but for the r2 instance.

            var r1 = new Random();
            var r2 = new Random();

            Debug.WriteLine("r1 sequence");
            for (int i = 0; i <5; i++)
            {
                Debug.WriteLine(r1.Next());
            }
            

            Debug.WriteLine("r2 sequence");
            for (int i = 0; i <5; i++)
            {
                Debug.WriteLine(r2.Next());
            }           

The r1 sequence is identical to the r2 instance, the same for the rest of the numbers in the sequence. This is because the 2 random instances have been created so close together that the same seed value was used from the system clock.

Below we're still creating effectively a sequence of 10 random numbers, but this time we're just using a single instance of the random class.

        [TestMethod]
        public void MoreRandomNumber()
        {
            var r1 = new Random();            

            Debug.WriteLine("r1 sequence");
            for (int i = 0; i < 5; i++)
            {
                Debug.WriteLine(r1.Next());
            }

            Debug.WriteLine("more r1 sequence");
            for (int i = 0; i < 5; i++)
            {
                Debug.WriteLine(r1.Next());
            }
        }

This will produce a better random sequence. The first five numbers here are different from the second five numbers here and this is because we're just using one instance of the random class. If you needed to generate random numbers between method calls then you could have a static random instance that's used by multiple methods. 

### Using Tuples to reduce code


Tuples are generic classes that we can use to hold sets of values of potentially different types. There's a couple of different ways we can create tuples. The first is to simply use the constructor. Below I'm creating a single element tuple using the new keyword specifying the type of int and giving it a value for the int. 

            var tupleOneElement = new Tuple<int>(1);

We can create 2 Element versions. Below is an int and a string

            var tupleTwoElement = new Tuple<int, string>(1, "hello");

and we can keep going all the way up to seven elements.

            var tupleSevenElement = 
                new Tuple<int, int, int, int, int, int, int>(1, 2, 3, 4, 5, 6, 7);

If we want to hold more than seven elements, we can do this, but the eighth element must be a tuple. Below is 8 elements, 1-7 with the eighth element being its own tuple and we can continue to nest these.
         
      var tupleEightElement =new Tuple<int, int, int, int, int, int, int, Tuple<string>>(1, 2, 3, 4, 5, 6, 7, new Tuple<string>("hello"));

The static Create method can be used to create Tuples. The Create method takes a list of values. Below we're creating a tuple with an integer, a string, and a DateTime.

            var tupleThreeElement = Tuple.Create(42, "hello", DateTime.Now);

Once we've created a tuple, we can access its properties using the .item1 or .item2 properties. If we created a 7 element tuple, we'd have item1 through to item7. 

            var t = Tuple.Create(42, "hello");

            int age = t.Item1;
            string greeting = t.Item2;

Once we create a tuple, we can't then change the values of the elements as tuples are immutable.

We can compare tuples. Below are 2 tuples with the same values, 42 and hello.
 
            var t1 = Tuple.Create(42, "hello");
            var t2 = Tuple.Create(42, "hello");

If we use the standard == operator, we get a reference comparison. So, isEqualTuples in this instance will be false as the variable t1 doesn't refer to the same thing as the variable t2.

         
            // Reference equality
            
            var isEqualTuples = t1 == t2;

If we want to get a value comparison, we can just use the Equals method. This will compare each individual element within each tuple. 

 
            // "value" comparison
            isEqualTuples = t1.Equals(t2);

A good use of tuples is to allow us to create composite keys in dictionaries. Below are three tuples and a SortedDictionary.

   
            var t1 = Tuple.Create(1, "z");
            var t2 = Tuple.Create(2, "a");
            var t3 = Tuple.Create(1, "a");

            var d = new SortedDictionary<Tuple<int, string>, string>();

The key in the dictionary is a tuple of int, string and the value in the dictionary is just a string. We then add our three tuples

         d.Add(t1, "this is Tuple t1");
         d.Add(t2, "this is Tuple t2");
         d.Add(t3, "this is Tuple t3");

The tuples appear in their sorted order. When sorting tuples, the first element is compared. If they are different, then that gives us our sort order. However, if they're the same, then we compare the next element in the tuple. If these are the same, then we'd compare the next, and so on.

### Simplify constructor overloads

Sometimes when we're writing our own classes, we want to provide a number of different overloads for the constructor. Doing so creates some flexibility in how the calling code creates one of the objects. 
In the code below, the Person class has three different overloads.
  
      class Person{
        private readonly string _name;
        private readonly int _age;
        private readonly string _gender;

        private const string DefaultGender = "Male";
        private const int DefaultAge = 30;     
        
        public Person(string name)
        {
            _name = name;
            _age = DefaultAge ;
            _gender = DefaultGender;
        }

        public Person(string name, int age)
        {
            _name = name;
            _age = age;
            _gender = DefaultGender;
        }

        public Person(string name, int age, string gender)
        {
            _name = name;
            _age = age;
            _gender = gender;
        }
	}

      [TestMethod]
      public void personConstrutors{
           var p1 = new Person("Lebron");
           var p1 = new Person("Kobi", 35);
           var p1 = new Person("Sir Charles", "Male", 50);
      }

p1 has some default age and some default gender with the name "Lebron".
P2 has the age that we passed, some default gender, and the name that we passed. 
p3 has the age, gender, and name that we passed. 

We can simplify this a bit and remove some code. We can do this by effectively calling one constructor from another one by using the This keyword. 

  
      class Person{
        private readonly string _name;
        private readonly int _age;
        private readonly string _gender;

        private const string DefaultGender = "Male";     
        private const int DefaultAge = 30;     

        public Person(string name)
            : this(name, DefaultAge, DefaultGender)
        {
        }

        public Person(string name, int age)
            : this(name, age, DefaultGender)
        {
        }

        public Person(string name, int age, string gender)
        {
            _name = name;
            _age = age;
            _gender = gender;
        }
    }

Chaining of constructors is a way that we can remove some redundant code.

### Static Array Methods

I've got an array of names

            var names = new[] { "Duncan", "James", "Bryant", "Robinson" };

There are a number of static methods of the Array class. 

#### Make an array ReadOnly 

If we ever need to provide a read-only view of an array, we can use the Array.AsReadOnly method. This method simply takes an array and returns a ReadOnlyCollection.

       [TestMethod]
        public void ReadonlyCollection()
        {
            var names = new[] { "Duncan", "James", "Bryant", "Robinson" };
            ReadOnlyCollection<string> ro = Array.AsReadOnly(names);
        }

#### Finding items in an array

To find the index of an item in the array, we can use the BinarySearchmethod or FindIndex method 

            int jamesIndex = Array.BinarySearch(names, "James");

            int jamesIndex = Array.FindIndex(names, x => x.StartsWith("James"));

If there are no matches, then we get a -1 returned. If there's more than one match, we get the index of the first matching item. 

#### Existence of items in an array
Use the Array.Exists method. 

            bool isDuncanThere = Array.Exists(names, x => x == "Duncan");

#### Find the last instance
Use the Array.FindLastmethod. 

            string lastBryant = Array.FindLast(names, x => x.StartsWith("Bryant"));

#### Iterate over an array
Use theArray.ForEach method. This takes an array and an action to be performed for each element in the array.

            Array.ForEach(names, x =>
                         {
                          Console.WriteLine(x);
                          C.WriteLine(x.Length);                                         
                         });

#### Sort an array
Use the Array.Sort method. 

            Array.Sort(names);

#### Test all elements in the array satisfy some condition.
Use the Array.TrueForAllmethod. This takes an array and a predicate that we're testing for.

            bool allNamesStartWithA = Array.TrueForAll(names, x => x.StartsWith("A"));

This returns false because not every element satisfies the predicate. 


#### Reverse an array
Use Array.Reverse method to reverse the items in the array.

            Array.Reverse(names);

When we're working with arrays, there's a number of different ways we can copy

#### Copying arrays

Use instance Clone method.
The Clone method returns an object, so we have to cast to an array of strings.

            string[] shallowCopy = (string[]) names.Clone();

            var originalElementZero = names[0];

            var copyElementZero = shallowCopy[0];


Our initial array contains four names. The shallowCopy has the same four names. When working with reference types in our array, we need to be aware that the Clone method creates a shallow copy of each item. This means that each element in the Source and Destination array point to the same place in memory. We can check this
   
            var isSameReference = object.ReferenceEquals(originalElementZero, copyElementZero);

So we've not performed a deep copy here. 
When using the Clone method with value types, we get a copy of each value.


CopyTo method allows us to copy the contents from one array to a Destinationarray. We can specify at which element in the Destination array to start copying to. 

Create an array of six elements of type string and populate the first four elements.

            var names = new string[6];
            names[0] = "Magic";
            names[1] = "Thomas";
            names[2] = "Jordan";
            names[3] = "Bird";

Create another array with another three names.

            var otherNames = new[] {"Ewing","Pipen","Drexler"};

Use the CopyTo command to copy from OtherNames array to names array. 

            otherNames.CopyTo(names, 3);

This populates our Destination Names array from element 3. So before we execute the CopyTo command, our Names array contains four names and two null values. Now after the CopyTo has executed, we can see that "Ewing", "Pipen" and "Drexler" have been copied from the otherNames array to the Names array and we've started overwriting the Names array at element 3. In this case "Bird" has been overwritten with "Ewing" and the original 2 null values have been replaced with "Pipen" and "Drexler". The same rules apply here for reference equality.

            var originalEwing = otherNames[0];
            var copiedEwing = names[3];

            var isSameReference =
                object.ReferenceEquals(originalMagic , copied);

We can see that Ewing in the original array points to the same place in memory as the copied Ewing because we're dealing with strings in this example, which are reference types.

##### Converting Arrays

To convert an array of one type to an array of a different type, we can use the array ConvertAllstatic method. 

            var ints = new[] {1, 2, 3 };

            BigInteger[] bigInts = Array.ConvertAll(ints, x => new BigInteger(x));

Each integer will be converted to a BigInteger. The resulting array now contains three BigIntegers, 1, 2, and 3. 

##### ConstrainedCopy

If we want to perform a copy from one array to another, but we want to ensure that the entire copy either succeeds or fails in an atomic fashion, we can use the array ConstrainedCopy method. This method takes the Source array and the start element of the Source array followed by the Destinationarray and the Destination element to start at and the number of elements we want to copy. 

            var things = new object[] { "Julius", 1};

            var thingsCopy = new object[2];            

            Array.ConstrainedCopy(things, 0, thingsCopy, 0, 2);

The above code copies from the array of things, which is an array of objects to the array called thingsCopy, which again is an array of objects. So here, each element in the Source array of things is convertible to each element of the Destination array of thingsCopy.

If we look at the result, we can see indeed that the copy was successful. However, if we declare a new array 

            var strings = new string[2];
            things.CopyTo(strings, 0);

This is an array of strings so the first element of the Source array, "Julius", will be convertible, but the second element, an integer, will not be. So if we execute 
            
we will get an exception because we can't cast from the integer of the second element to a string. But the first element wiil be populated where as the second element, where this exception occurred, will not be. The Destination strings array is in an inconsistent state. 

This is where the ConstrainedCopy method can be used to handle this situation. 

            var strings = new string[2];
            Array.ConstrainedCopy(things, 0, strings, 0, 2);

Again, we get an exception, but this time if we look at ourDestination array, we can see that the first element is no longer "Julius" and that's because ConstrainedCopywill either copy all elements successfully or it will roll back the initial state of the Destination array.


### Converting base types to byte arrays

If we're working with some of the base types in C#, we can make use of the BitConverter class to convert these to and from arrays of byte values. Sometimes this is useful if we need to get binary representations of values. The BitConverter class contains a static GetByte method. 
BitConverter.GetByte 
This allows us to pass the value to be converted to a byteArray. The GetBytes method supports Booleans, characters, doubles, floats,integers, longs, shorts, unsigned integers, unsigned longs, and unsigned shorts.


            const float originalFloat = 24.56f;

            byte[] byteArray = BitConverter.GetBytes(originalFloat);

In the example here, we're just setting up an original Float value, 24.56. We're then using the BitConverter.GetBytes method to create an array of bytes.

To convert this array of bytes back to the originalFloat value, we can use theBitConverter.To Single method. 

float convertedBack = BitConverter.ToSingle(byteArray, 0);

The BitConverter also contains a number of methods such as ToBoolean, ToDouble, ToInt16, 32, 64, and so on, to allow us to convert back from an array of bytes to the original value. In addition to providing the array, we also have to tell the method at which byte in the array to start. In the above example, the array only contains the originalFloat, so we tell it to start at the beginning.
So the BitConverter class allows us to get the byte values for some of the basic types. The GetBytes method doesn't allow us to serialize all of the types. For example, it doesn't support DateTimes, but we're still able to convert DateTimes to an array of bytes. The first step is to make use of the DateTimes ToBinary method. The ToBinary method gives us a binary value for the DateTime and gives us a long. Once we have a long, we can then use the BitConverter.GetByte method as this supports long values. 

            long originalLong = originalDate.ToBinary();

            byte[] byteArray = BitConverter.GetBytes(originalLong);

So here we get back an array of bytes whuch can use the DateTime.FromBinary method, which expects a long.

To convert from our array of bytes, we use the BitConverter.ToInt64 method, passing a bytearray that contains an Int64 and again we tell it to start at the beginning of the array.

	       DateTime convertedBack = DateTime.FromBinary(BitConverter.ToInt64(byteArray, 0));


