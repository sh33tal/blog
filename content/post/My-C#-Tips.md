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

            var name = "Sarah";

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

We can create TwoElement versions. Below is an int and a string

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