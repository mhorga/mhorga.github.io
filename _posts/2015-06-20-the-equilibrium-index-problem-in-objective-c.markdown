---
published: true
title: The Equilibrium Index problem in Objective-C
layout: post
---
One of the demo challenges at __Codility.com__ is the [Equilibrium Index](https://codility.com/demo/take-sample-test/) problem. Since I noticed in the solution feedback that there were few complaints about the Objective-C online compiler not working I decided to give it a try myself. The problem description follows.

The _equilibrium index_ of a sequence is an index such that the sum of elements at lower indexes is equal to the sum of elements at higher indexes. For example, in a sequence A:

`A[0]=-7 A[1]=1 A[2]=5 A[3]=2 A[4]=-4 A[5]=3 A[6]=0`

3 is an equilibrium index, because:

`A[0]+A[1]+A[2]=A[4]+A[5]+A[6]`

6 is also an equilibrium index, because:

`A[0]+A[1]+A[2]+A[3]+A[4]+A[5]=0`

(The sum of zero elements is zero) so 7 is not an equilibrium index because it is not a valid index of sequence A.

Your challenge is to write a function __int equilibrium(int A[])__ that, given a sequence, returns its equilibrium index (any) or -1 if no equilibrium index exists. Assume that the sequence may be very long. The problem can be solved by using various approaches, the most common being simply to follow the equilibrium definition. Create an empty project and use the __main.m__ file to run the code:

```objective-c
int equilibrium(NSMutableArray *A) {
    int i, j, equi = -1;
    for (i=0; i<A.count; ++i) {
        int lsum = 0, rsum = 0;
        for (j=0; j<i; ++j) {
            lsum += [A[j] integerValue];
        }
        for (j=i+1; j<A.count; ++j) {
            rsum += [A[j] integerValue];
        }
        if (lsum == rsum) {
            equi = i;
            NSLog(@"%d ", equi);
        }
    }
    return equi;
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSMutableArray *array = [[NSMutableArray alloc] initWithArray:@[@-1, @3, @-4, @5, @1, @-6, @2, @1]];
        NSLog(@"Equilibrium points:");
        equilibrium(array);
    }
    return 0;
} 
```

Here is the score I got using this approach:
![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/equi_bad.png "Bad score")

It seems this approach was not efficient for two reasons:

- it takes way too long to process large input data sets because time complexity is quadratic or O(n^2)
 
- it fails on large input values (outside the __int__ min/max limits) due to the arithmetic overflows

We can improve our algorithm by updating the left/right sums in O(0) time instead of recomputing them again at each iteration. To handle larger input values we should use a proper data-type such as __long long__ instead of __int__. Here is a better solution:

```
int equilibrium(NSMutableArray *A) {

    long long sum = 0;
    
    int i, equi = -1;
    
    for(i=0; i<A.count; i++) {
    
        sum += (long long)[A[i] integerValue];
        
    }
    
    long long sum_left = 0;
    
    for(i=0;i<A.count;i++) {
    
        long long sum_right = sum - sum_left - (long long)[A[i] integerValue];
        
        if (sum_left == sum_right) {
        
            equi = i;
            
            NSLog(@"%d", i);
            
        }
        
        sum_left += (long long)[A[i] integerValue];
        
    }
    
    return equi;
    
}

```

Using this solution we get perfect score:
![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/equi_good.png "Good score")

Until next time!
