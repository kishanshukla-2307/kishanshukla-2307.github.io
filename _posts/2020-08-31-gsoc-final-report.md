---
title: 'Google Summer of Code 2020'
date: 2020-08-31
permalink: /posts/2020/08/gsoc-final-evaluation/
tags:
  - Google Summer of Code 2020
  - Boost C++
  - Boost.Real
  - C++17
  - arbitrary-precision arithmetic
  - numerical computing
---

# Bringing Boost.Real to review-ready state
<img src='/images/boost_logo.png' alt="boost_logo.png" style="horizontal-align:middle;margin:0px 50px">

## Boost.Real Documentation
- [Project Documentation](https://boostgsoc18.github.io/Real/doc/html/index.html)  

## Project Details
- __Source__ __Code:__ [https://github.com/BoostGSoC20/Real](https://github.com/BoostGSoC20/Real).<br>
- __Foundational idea behind Boost.Real:__ [Blog](https://medium.com/@laobelloli/boost-real-9e2dfbfbed5b) by Laouen Belloli
- __Contributors:__ Laouen Belloli (author), Sagnik Dey, Kimberly Swanson, Kishan Shukla, Vikram Chundawat.<br>
- __Mentors:__ Damian Vicino, Laouen Belloli.<br>
- __Previous__ __work:__ <br>
	- [https://medium.com/@laobelloli/boost-real-9e2dfbfbed5b](https://medium.com/@laobelloli/boost-real-9e2dfbfbed5b) (work done by Laouen Belloli during GSoC'18 under the guidance of Damian Vicino)
	- [https://sagnikdey92.github.io/GSoC](https://sagnikdey92.github.io/GSoC) (work done by Sagnik Dey during GSoC'19 under the guidance of Damian Vicino and Laouen Belloli)
	- [https://universenox.github.io/gsoc19_Final_Eval](https://universenox.github.io/gsoc19_Final_Eval) (work done by Kimberly Swanson during GSoC'19 under the guidance of Damian Vicino and Laouen Belloli)
- __Current__ __work:__ <br>
	- This is the report of my work during GSoC'20 under the guidance of Damian Vicino and Laouen Belloli.
	- [https://medium.com/@vikram2000b/56b2582773d3](https://medium.com/@vikram2000b/56b2582773d3) Report of Vikram's work during GSoC'20 under the guidance of Damian Vicino and Laouen Belloli.

## Introduction
__Boost.Real__ is a C++17 library which aims at developing numerical data-type for real number representation. The library provides the flexibility of performing __arbitrary-precision__ __arithmetic__. The foundational work was done by __Laouen__ __belloli__ in GSoC'18, after that, several improvements and new additions were made in GSoC'19 by Sagnik Dey and Kimberly Swanson. For GSoC'20, I and Vikram Chundawat were selected to finish the details and add few functionalities and hence make the library review-ready. Since Vikram and mine's proposal had several overlaps, we distributed the work among ourselves. After the distribution, I got following tasks: <br>
- Changing internal base of Boost.Real numbers to a more efficient value.
- Implementation of optimized long division algorithm.
- Implementation of division operator.
- Implementation of integral power operation.
- Implementation of method to evaluate Pi digits.
- Implementaion of % operator.
- Implementation of Karatsuba Multiplication algorithm.
- Implementation of tests and documentation.  

Before the start of official coding period, we decided on not changing the base of the internal representation as changing it would make few operations more expensive resulting in poor performance.

## Phase I

__Milestones:__<br>
- Implementation of long division algorithm.
- Implementation of division operator.
- Implementation of integral power operation.
- Adding tests and documentation.

### Long division algorithm
Long division was already implemented in the library for base 10, since it was required for the purpose of base change. I had to implement it for any general base as I was going to need it while implementing % operator.  
I used __Knuth's long division algorithm__ as it is quite efficient and easy to implement as well. The Knuth's algorithm optimizes the part where we search for quotient digit (can be anything between 0 to base - 1) in the traditional algorithm. In the traditional algorithm, the search is performed linearly from 0 to Base - 1 making the complexity O(base) (a naive optimization would be to use binary search) whereas the Knuth's algorithm does this in constant time, which provides significant optimization as our internal base increases (our internal base can go upto 10<sup>18</sup>).   
Initial Commit regarding Knuth's long division: [https://github.com/BoostGSoC20/Real/commit/25f1a06a7ea91bed2413a6417f842ed10dee7c3a](https://github.com/BoostGSoC20/Real/commit/25f1a06a7ea91bed2413a6417f842ed10dee7c3a)  
Refrence to Knuth's Algorithm: algorithm D of section 4.3.2 of volume 2 of __The Art of Computer Programming by D. E. Knuth__.  

### Division Operator
This was the most important task in bringing the library to review-ready state. I used Newton-Raphson method for the division algorithm, it involves evaluating reciprocal of the divisor and then multiplying it with the dividend to get the result. The method is quite efficient because of the quadratic convergence of the Newton-Raphson algorithm. The initial guess that I used for Newton-Raphson algorithm involved division therefore I implemented an approximate division algorithm to evaluate the initial guess. Later on the initial guess can replaced with something that doesn't involve division.  
Intial commit regarding division operator: [https://github.com/BoostGSoC20/Real/commit/599890e5fdf9290bb1197d7b054d0517a18f58a3](https://github.com/BoostGSoC20/Real/commit/599890e5fdf9290bb1197d7b054d0517a18f58a3)  

### Integral Power Operation
Integral Power was implemented as new operation between two reals. The implementation was based on const_precision_iterators which makes it more general as were the other operations already present. Binary exponentiation was used to evaluate the powers, making the complexity O(C * log(exponent)) where C is the time taken to multiply two reals.  
Following is overview of how power operation has been implemented (few lines of code has been skipped) :  

<pre style='color:#000000;background:#F5F5F5;font-size:9pt'><span style='color:#800000; font-weight:bold; '>case </span><span style='color:#7d0045; '>OPERATION</span><span style='color:#800080; '>::</span><span style='color:#7d0045; '>INTEGER_POWER</span><span style='color:#e34adc; '>:</span> <span style='color:#800080; '>{</span>
	ro<span style='color:#808030; '>.</span>get_rhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>iterate_n_times<span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_rhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>maximum_precision<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>

	<span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_rhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span> <span style='color:#808030; '>!</span><span style='color:#808030; '>=</span> ro<span style='color:#808030; '>.</span>get_rhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>upper_bound</span> <span style='color:#808030; '>|</span><span style='color:#808030; '>|</span>
	<span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>int</span><span style='color:#808030; '>)</span> ro<span style='color:#808030; '>.</span>get_rhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span><span style='color:#808030; '>.</span>digits<span style='color:#808030; '>.</span>size<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>></span> ro<span style='color:#808030; '>.</span>get_rhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span><span style='color:#808030; '>.</span>exponent<span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
		<span style='color:#800000; font-weight:bold; '>throw</span> non_integral_exponent_exception<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800080; '>}</span>

	<span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_rhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>upper_bound</span><span style='color:#808030; '>.</span>positive <span style='color:#808030; '>=</span><span style='color:#808030; '>=</span> <span style='color:#800000; font-weight:bold; '>false</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
		<span style='color:#800000; font-weight:bold; '>throw</span> negative_integers_not_supported<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800080; '>}</span>

	<span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_lhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>positive<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
		<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>_approximation_interval<span style='color:#808030; '>.</span><span style='color:#603000; '>upper_bound</span> <span style='color:#808030; '>=</span> 
			tmp<span style='color:#808030; '>.</span>binary_exponentiation<span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_lhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>upper_bound</span><span style='color:#808030; '>,</span> exponent<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
		<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>_approximation_interval<span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span> <span style='color:#808030; '>=</span>
			tmp<span style='color:#808030; '>.</span>binary_exponentiation<span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_lhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span><span style='color:#808030; '>,</span> exponent<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800080; '>}</span> <span style='color:#800000; font-weight:bold; '>else</span> <span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_lhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>negative<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
		<span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>exponent_is_even<span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
			<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>_approximation_interval<span style='color:#808030; '>.</span><span style='color:#603000; '>upper_bound</span> <span style='color:#808030; '>=</span>
			    tmp<span style='color:#808030; '>.</span>binary_exponentiation<span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_lhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span><span style='color:#808030; '>,</span> exponent<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
			<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>_approximation_interval<span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span> <span style='color:#808030; '>=</span>
			    tmp<span style='color:#808030; '>.</span>binary_exponentiation<span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_lhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>upper_bound</span><span style='color:#808030; '>,</span> exponent<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
		<span style='color:#800080; '>}</span> <span style='color:#800000; font-weight:bold; '>else</span> <span style='color:#800080; '>{</span>
			<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>_approximation_interval<span style='color:#808030; '>.</span><span style='color:#603000; '>upper_bound</span> <span style='color:#808030; '>=</span>
			    tmp<span style='color:#808030; '>.</span>binary_exponentiation<span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_lhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>upper_bound</span><span style='color:#808030; '>,</span> exponent<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
			<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>_approximation_interval<span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span> <span style='color:#808030; '>=</span>
			    tmp<span style='color:#808030; '>.</span>binary_exponentiation<span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_lhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span><span style='color:#808030; '>,</span> exponent<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
		<span style='color:#800080; '>}</span>
	<span style='color:#800080; '>}</span> <span style='color:#800000; font-weight:bold; '>else</span> <span style='color:#800080; '>{</span>
		<span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>exponent_is_even<span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
			<span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_lhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>upper_bound</span><span style='color:#808030; '>.</span>abs<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>></span> ro<span style='color:#808030; '>.</span>get_lhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span><span style='color:#808030; '>.</span>abs<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
				<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>_approximation_interval<span style='color:#808030; '>.</span><span style='color:#603000; '>upper_bound</span> <span style='color:#808030; '>=</span>
					tmp<span style='color:#808030; '>.</span>binary_exponentiation<span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_lhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>upper_bound</span><span style='color:#808030; '>,</span> exponent<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
				<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>_approximation_interval<span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span> <span style='color:#808030; '>=</span> zero<span style='color:#800080; '>;</span>
			<span style='color:#800080; '>}</span> <span style='color:#800000; font-weight:bold; '>else</span> <span style='color:#800080; '>{</span>
				<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>_approximation_interval<span style='color:#808030; '>.</span><span style='color:#603000; '>upper_bound</span> <span style='color:#808030; '>=</span>
					tmp<span style='color:#808030; '>.</span>binary_exponentiation<span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_lhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span><span style='color:#808030; '>,</span> exponent<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
				<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>_approximation_interval<span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span> <span style='color:#808030; '>=</span> zero<span style='color:#800080; '>;</span>
			<span style='color:#800080; '>}</span>
		<span style='color:#800080; '>}</span> <span style='color:#800000; font-weight:bold; '>else</span> <span style='color:#800080; '>{</span>
			<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>_approximation_interval<span style='color:#808030; '>.</span><span style='color:#603000; '>upper_bound</span> <span style='color:#808030; '>=</span>
				tmp<span style='color:#808030; '>.</span>binary_exponentiation<span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_lhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>upper_bound</span><span style='color:#808030; '>,</span> exponent<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
			<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>_approximation_interval<span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span> <span style='color:#808030; '>=</span>
				tmp<span style='color:#808030; '>.</span>binary_exponentiation<span style='color:#808030; '>(</span>ro<span style='color:#808030; '>.</span>get_lhs_itr<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span>get_interval<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>.</span><span style='color:#603000; '>lower_bound</span><span style='color:#808030; '>,</span> exponent<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
		<span style='color:#800080; '>}</span>
	<span style='color:#800080; '>}</span>

	<span style='color:#800000; font-weight:bold; '>break</span><span style='color:#800080; '>;</span>
<span style='color:#800080; '>}</span> 
</pre>
<!--Created using ToHtml.com on 2020-08-29 06:39:38 UTC -->



Intial commit regarding integral power operation: [https://github.com/BoostGSoC20/Real/commit/84f36e9e2451f15d1ebf42178944e91eb34b7147](https://github.com/BoostGSoC20/Real/commit/84f36e9e2451f15d1ebf42178944e91eb34b7147)  


## Phase II 

__Milestones:__<br>
- Implementing method to evaluate digits of Pi.
- Implementing % operator.
- Implementing calculation of few digits of real_algorithm number during compile time 

### Pi
Pi digits were calculated using Chudnovsky algorithm. The implementation is not the most efficient one therefore after few digits the algorithm runs slow. Chudnovsky algorithm is an iterative algorithm which is being used to calculate the n<sup>th</sup> digit of the real_algorithm number. There are redundant calculations being performed as for calculating (n + 1)<sup>th</sup> digit, the algorithm recalculates the previously calculated values as this time it needs more precise values than previous. There is an algorithm which calculates n<sup>th</sup> digit without calculating previous digits, but the algorithm gives output in hexadecimal base, which doesn't fulfill our requirement.  
Following is overview of how Chudnovsky algorithm has been implemented (few lines of code has been skipped) :  

<pre style='color:#000000;background:#F5F5F5;font-size:9pt'><span style='color:#800000; font-weight:bold; '>template</span> <span style='color:#800080; '>&lt;</span><span style='color:#800000; font-weight:bold; '>typename</span> T <span style='color:#808030; '>=</span> <span style='color:#800000; font-weight:bold; '>int</span><span style='color:#800080; '>></span>
T pi_nth_digit<span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>unsigned</span> <span style='color:#800000; font-weight:bold; '>int</span> n<span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
	<span style='color:#696969; '>// Chudnovsky Algorithm</span>
	<span style='color:#696969; '>// pi = C * ( sum_from_k=0_to_k=x (Mk * Lk / Xk) )^(-1) </span>
	<span style='color:#696969; '>// increasing x you get more precise pi</span>

	<span style='color:#800000; font-weight:bold; '>static</span> <span style='color:#800000; font-weight:bold; '>const</span> boost<span style='color:#800080; '>::</span>real<span style='color:#800080; '>::</span>real_explicit<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> real_k<span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>6</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>static</span> <span style='color:#800000; font-weight:bold; '>const</span> boost<span style='color:#800080; '>::</span>real<span style='color:#800080; '>::</span>real_explicit<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> real_m<span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>1</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>static</span> <span style='color:#800000; font-weight:bold; '>const</span> boost<span style='color:#800080; '>::</span>real<span style='color:#800080; '>::</span>real_explicit<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> real_l<span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>13591409</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>static</span> <span style='color:#800000; font-weight:bold; '>const</span> boost<span style='color:#800080; '>::</span>real<span style='color:#800080; '>::</span>real_explicit<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> real_l0<span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>545140134</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>static</span> <span style='color:#800000; font-weight:bold; '>const</span> boost<span style='color:#800080; '>::</span>real<span style='color:#800080; '>::</span>real_explicit<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> real_x<span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>1</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>static</span> <span style='color:#800000; font-weight:bold; '>const</span> boost<span style='color:#800080; '>::</span>real<span style='color:#800080; '>::</span>real_explicit<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> real_x0<span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>-262537412640768000</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>static</span> <span style='color:#800000; font-weight:bold; '>const</span> boost<span style='color:#800080; '>::</span>real<span style='color:#800080; '>::</span>real_explicit<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> real_s<span style='color:#808030; '>(</span><span style='color:#800000; '>"</span><span style='color:#0000e6; '>13591409</span><span style='color:#800000; '>"</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>

	<span style='color:#800000; font-weight:bold; '>static</span> exact_number<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> L0 <span style='color:#808030; '>=</span> real_l0<span style='color:#808030; '>.</span>get_exact_number<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>static</span> exact_number<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> X0 <span style='color:#808030; '>=</span> real_x0<span style='color:#808030; '>.</span>get_exact_number<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>

	<span style='color:#800000; font-weight:bold; '>bool</span> nth_digit_found <span style='color:#808030; '>=</span> <span style='color:#800000; font-weight:bold; '>false</span><span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>bool</span> first_iteration_over <span style='color:#808030; '>=</span> <span style='color:#800000; font-weight:bold; '>false</span><span style='color:#800080; '>;</span>

	exact_number<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> iteration_number <span style='color:#808030; '>=</span> one<span style='color:#800080; '>;</span>
	exact_number<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> prev_pi<span style='color:#800080; '>;</span>
	exact_number<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> pi<span style='color:#800080; '>;</span>
	exact_number<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> error<span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>const</span> exact_number<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> max_error<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#ffffff; font-weight:bold; font-style:italic; '>{</span><span style='color:#008c00; '>1</span><span style='color:#ffffff; font-weight:bold; font-style:italic; '>}</span><span style='color:#808030; '>,</span> <span style='color:#808030; '>-</span><span style='color:#808030; '>(</span>n <span style='color:#808030; '>+</span> <span style='color:#008c00; '>1</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>

	<span style='color:#800000; font-weight:bold; '>do</span> <span style='color:#800080; '>{</span>  
	    exact_number<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> temp <span style='color:#808030; '>=</span> K <span style='color:#808030; '>*</span> K <span style='color:#808030; '>*</span> K <span style='color:#808030; '>-</span> _16 <span style='color:#808030; '>*</span> K<span style='color:#800080; '>;</span>
	    temp<span style='color:#808030; '>.</span>divide_vector<span style='color:#808030; '>(</span>iteration_number <span style='color:#808030; '>*</span> iteration_number <span style='color:#808030; '>*</span> iteration_number<span style='color:#808030; '>,</span> n <span style='color:#808030; '>+</span> <span style='color:#008c00; '>1</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	    M <span style='color:#808030; '>*</span><span style='color:#808030; '>=</span> temp<span style='color:#800080; '>;</span>
	    X <span style='color:#808030; '>*</span><span style='color:#808030; '>=</span> X0<span style='color:#800080; '>;</span>
	    L <span style='color:#808030; '>+</span><span style='color:#808030; '>=</span> L0<span style='color:#800080; '>;</span>
	    K <span style='color:#808030; '>+</span><span style='color:#808030; '>=</span> _12<span style='color:#800080; '>;</span>

	    temp <span style='color:#808030; '>=</span> M <span style='color:#808030; '>*</span> L<span style='color:#800080; '>;</span>
	    temp<span style='color:#808030; '>.</span>divide_vector<span style='color:#808030; '>(</span>X<span style='color:#808030; '>,</span> n <span style='color:#808030; '>+</span> <span style='color:#008c00; '>1</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>false</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	    S <span style='color:#808030; '>+</span><span style='color:#808030; '>=</span> temp<span style='color:#800080; '>;</span>

	    temp <span style='color:#808030; '>=</span> C<span style='color:#800080; '>;</span>
	    temp<span style='color:#808030; '>.</span>divide_vector<span style='color:#808030; '>(</span>S<span style='color:#808030; '>,</span> n <span style='color:#808030; '>+</span> <span style='color:#008c00; '>1</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>false</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>

	    pi <span style='color:#808030; '>=</span> temp<span style='color:#800080; '>;</span>
	    <span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span><span style='color:#808030; '>!</span>first_iteration_over<span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
		prev_pi <span style='color:#808030; '>=</span> pi<span style='color:#800080; '>;</span>
		first_iteration_over <span style='color:#808030; '>=</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#800080; '>;</span>
		iteration_number <span style='color:#808030; '>+</span><span style='color:#808030; '>=</span> one<span style='color:#800080; '>;</span>
	    <span style='color:#800080; '>}</span> <span style='color:#800000; font-weight:bold; '>else</span> <span style='color:#800080; '>{</span>
		error <span style='color:#808030; '>=</span> pi <span style='color:#808030; '>-</span> prev_pi<span style='color:#800080; '>;</span>
		error<span style='color:#808030; '>.</span>positive <span style='color:#808030; '>=</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#800080; '>;</span>

		<span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>error <span style='color:#808030; '>&lt;</span> max_error<span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
		    nth_digit_found <span style='color:#808030; '>=</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#800080; '>;</span>
		<span style='color:#800080; '>}</span>
		iteration_number <span style='color:#808030; '>+</span><span style='color:#808030; '>=</span> one<span style='color:#800080; '>;</span>
		prev_pi <span style='color:#808030; '>=</span> pi<span style='color:#800080; '>;</span>
	    <span style='color:#800080; '>}</span>

	<span style='color:#800080; '>}</span> <span style='color:#800000; font-weight:bold; '>while</span> <span style='color:#808030; '>(</span><span style='color:#808030; '>!</span>nth_digit_found<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>

	<span style='color:#800000; font-weight:bold; '>return</span> pi<span style='color:#808030; '>[</span>n<span style='color:#808030; '>]</span><span style='color:#800080; '>;</span>
<span style='color:#800080; '>}</span>
</pre>
<!--Created using ToHtml.com on 2020-08-29 06:50:35 UTC -->

Intial commit regarding Pi: [https://github.com/BoostGSoC20/Real/commit/34ac759f128cd2ffb5d684c5ff7a5a0e54025360](https://github.com/BoostGSoC20/Real/commit/34ac759f128cd2ffb5d684c5ff7a5a0e54025360)  

### % operator
As % operator is only defined for integers, this operator was overloaded by Vikram in his integer_number class. The operator used the Knuth's long division algorithm to find the remainder.

### Calculation of digits of real_algorithm number during compile-time
This task was proposed to provide an optimization for real_algorithm numbers. The task involved calculating the starting few digits of real_algorithm number during compilation, if the function for n'th digit provided by the user is constexpr. It seemed that the task wasn't doable since it was not possible to detect whether the function provided by the user is constepxr or not. (In normal scenarios it is possible to check whether the function is constexpr or not, but in our case the since the was passed between many functions before the check was performed it's constexpr'ness is lost in most of the cases.)  


## Phase III

__Milestone:__<br>
- Implement Karatsuba algorithm.
- Tests and Documentation.  

### Karatsuba algorithm
I implemented the Karatsuba algorithm which improved the performance of the multiplication operation for vectors of much bigger sizes. The implementation is bit more optimised than the traditional implementation as we are not storing zeroes (required to make the length of operands equal) in vectors. This small optimisation saves a lot of memory as well as time. Benchmarking of the algorithm was done by multiplying incrementally larger examples using both standard multiplication algorithm and Karatsuba algorithm and comparing the performances. The examples used for the benchmarking purpose were vectors of same length, in general the benchmarking becomes difficult if we consider examples of different sizes which is more practical situation.  

Following is overview of how karatsuba algorithm has been implemented (few lines of code has been skipped) :  

<pre style='color:#000000;background:#F5F5F5;font-size:9pt'><span style='color:#800000; font-weight:bold; '>template</span> <span style='color:#800080; '>&lt;</span><span style='color:#800000; font-weight:bold; '>typename</span> T <span style='color:#808030; '>=</span> <span style='color:#800000; font-weight:bold; '>int</span><span style='color:#800080; '>></span>
<span style='color:#800000; font-weight:bold; '>void</span> karatsuba_multiplication <span style='color:#808030; '>(</span>
    exact_number<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>&amp;</span>other<span style='color:#808030; '>,</span> 
    <span style='color:#800000; font-weight:bold; '>const</span> T base <span style='color:#808030; '>=</span> <span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span>numeric_limits<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span><span style='color:#800080; '>::</span><span style='color:#603000; '>max</span><span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>/</span> <span style='color:#008c00; '>4</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>*</span> <span style='color:#008c00; '>2</span>
<span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>

	<span style='color:#696969; '>// this --- a, other --- b</span>
	<span style='color:#800000; font-weight:bold; '>const</span> <span style='color:#800000; font-weight:bold; '>int</span> a_size <span style='color:#808030; '>=</span> <span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>digits<span style='color:#808030; '>.</span>size<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>const</span> <span style='color:#800000; font-weight:bold; '>int</span> b_size <span style='color:#808030; '>=</span> other<span style='color:#808030; '>.</span>digits<span style='color:#808030; '>.</span>size<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>const</span> <span style='color:#800000; font-weight:bold; '>int</span> a_exponent <span style='color:#808030; '>=</span> <span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>exponent<span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>const</span> <span style='color:#800000; font-weight:bold; '>int</span> b_exponent <span style='color:#808030; '>=</span> other<span style='color:#808030; '>.</span>exponent<span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>const</span> <span style='color:#800000; font-weight:bold; '>bool</span> a_sign <span style='color:#808030; '>=</span> <span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>positive<span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>const</span> <span style='color:#800000; font-weight:bold; '>bool</span> b_sign <span style='color:#808030; '>=</span> other<span style='color:#808030; '>.</span>positive<span style='color:#800080; '>;</span>

	<span style='color:#800000; font-weight:bold; '>const</span> <span style='color:#800000; font-weight:bold; '>int</span> max_length <span style='color:#808030; '>=</span> <span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>max</span><span style='color:#808030; '>(</span>a_size<span style='color:#808030; '>,</span> b_size<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>

	<span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>max_length <span style='color:#808030; '>&lt;</span><span style='color:#808030; '>=</span> KARATSUBA_BASE_CASE_THRESHOLD <span style='color:#808030; '>|</span><span style='color:#808030; '>|</span> <span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>abs</span><span style='color:#808030; '>(</span>a_size <span style='color:#808030; '>-</span> b_size<span style='color:#808030; '>)</span> <span style='color:#808030; '>></span> <span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>min</span><span style='color:#808030; '>(</span>a_size<span style='color:#808030; '>,</span> b_size<span style='color:#808030; '>)</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
	    <span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>standard_multiplication<span style='color:#808030; '>(</span>other<span style='color:#808030; '>,</span> base<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	    <span style='color:#800000; font-weight:bold; '>return</span><span style='color:#800080; '>;</span>
	<span style='color:#800080; '>}</span>

	<span style='color:#696969; '>// appending zeroes in front to make sizes of a &amp; b equal</span>
	<span style='color:#800000; font-weight:bold; '>int</span> a_pref_zeroes <span style='color:#808030; '>=</span> <span style='color:#008c00; '>0</span><span style='color:#808030; '>,</span> b_pref_zeroes <span style='color:#808030; '>=</span> <span style='color:#008c00; '>0</span><span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>a_size <span style='color:#808030; '>&lt;</span> max_length<span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
	    a_pref_zeroes <span style='color:#808030; '>=</span> max_length <span style='color:#808030; '>-</span> a_size<span style='color:#800080; '>;</span>
	<span style='color:#800080; '>}</span> <span style='color:#800000; font-weight:bold; '>else</span> <span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>b_size <span style='color:#808030; '>&lt;</span> max_length<span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
	    b_pref_zeroes <span style='color:#808030; '>=</span> max_length <span style='color:#808030; '>-</span> b_size<span style='color:#800080; '>;</span>
	<span style='color:#800080; '>}</span> 

	<span style='color:#800000; font-weight:bold; '>const</span> <span style='color:#800000; font-weight:bold; '>int</span> left_half_length <span style='color:#808030; '>=</span> max_length <span style='color:#808030; '>/</span> <span style='color:#008c00; '>2</span><span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>const</span> <span style='color:#800000; font-weight:bold; '>int</span> right_half_length <span style='color:#808030; '>=</span> max_length <span style='color:#808030; '>-</span> left_half_length<span style='color:#800080; '>;</span>

	<span style='color:#696969; '>/*</span>
<span style='color:#696969; '>		Variable Explanation</span>
<span style='color:#696969; '>	    a is vector representation of "this", b is vector representation of "other"</span>
<span style='color:#696969; '>	    a = exact_al * base^(right_half_length) + exact_ar</span>
<span style='color:#696969; '>	    b = exact_bl * base^(right_half_length) + exact_br</span>
<span style='color:#696969; '>	*/</span>
	
	<span style='color:#696969; '>// This is where we perform optimization (not appending zeroes in vector)</span>
	<span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>a_pref_zeroes <span style='color:#808030; '>></span> <span style='color:#008c00; '>0</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
	    <span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>a_pref_zeroes <span style='color:#808030; '>></span><span style='color:#808030; '>=</span> left_half_length<span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
		exact_al <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
		exact_ar <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>digits<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	    <span style='color:#800080; '>}</span> <span style='color:#800000; font-weight:bold; '>else</span> <span style='color:#800080; '>{</span>
		exact_al <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>+</span> left_half_length <span style='color:#808030; '>-</span> a_pref_zeroes<span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
		exact_ar <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>+</span> left_half_length <span style='color:#808030; '>-</span> a_pref_zeroes<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>digits<span style='color:#808030; '>.</span>end<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	    <span style='color:#800080; '>}</span>
	    exact_bl <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>(</span>other<span style='color:#808030; '>.</span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> other<span style='color:#808030; '>.</span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>+</span> left_half_length<span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	    exact_br <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>(</span>other<span style='color:#808030; '>.</span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>+</span> left_half_length<span style='color:#808030; '>,</span> other<span style='color:#808030; '>.</span>digits<span style='color:#808030; '>.</span>end<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800080; '>}</span> <span style='color:#800000; font-weight:bold; '>else</span> <span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>b_pref_zeroes <span style='color:#808030; '>></span> <span style='color:#008c00; '>0</span><span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
	    <span style='color:#800000; font-weight:bold; '>if</span> <span style='color:#808030; '>(</span>b_pref_zeroes <span style='color:#808030; '>></span><span style='color:#808030; '>=</span> left_half_length<span style='color:#808030; '>)</span> <span style='color:#800080; '>{</span>
		exact_bl <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
		exact_br <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span>other<span style='color:#808030; '>.</span>digits<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	    <span style='color:#800080; '>}</span> <span style='color:#800000; font-weight:bold; '>else</span> <span style='color:#800080; '>{</span>
		exact_bl <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>(</span>other<span style='color:#808030; '>.</span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> other<span style='color:#808030; '>.</span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>+</span> left_half_length <span style='color:#808030; '>-</span> b_pref_zeroes<span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
		exact_br <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>(</span>other<span style='color:#808030; '>.</span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>+</span> left_half_length <span style='color:#808030; '>-</span> b_pref_zeroes<span style='color:#808030; '>,</span> other<span style='color:#808030; '>.</span>digits<span style='color:#808030; '>.</span>end<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	    <span style='color:#800080; '>}</span>
	    exact_al <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>+</span> left_half_length<span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	    exact_ar <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>+</span> left_half_length<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>digits<span style='color:#808030; '>.</span>end<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800080; '>}</span> <span style='color:#800000; font-weight:bold; '>else</span> <span style='color:#800080; '>{</span>
	    exact_al <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>+</span> left_half_length<span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	    exact_ar <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>(</span><span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>+</span> left_half_length<span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>digits<span style='color:#808030; '>.</span>end<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	    exact_bl <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>(</span>other<span style='color:#808030; '>.</span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> other<span style='color:#808030; '>.</span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>+</span> left_half_length<span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	    exact_br <span style='color:#808030; '>=</span> exact_number<span style='color:#808030; '>(</span><span style='color:#666616; '>std</span><span style='color:#800080; '>::</span><span style='color:#603000; '>vector</span><span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> <span style='color:#808030; '>(</span>other<span style='color:#808030; '>.</span>digits<span style='color:#808030; '>.</span>begin<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span> <span style='color:#808030; '>+</span> left_half_length<span style='color:#808030; '>,</span> other<span style='color:#808030; '>.</span>digits<span style='color:#808030; '>.</span>end<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#808030; '>)</span><span style='color:#808030; '>,</span> <span style='color:#800000; font-weight:bold; '>true</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800080; '>}</span>

	exact_number<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> sum_al_ar <span style='color:#808030; '>(</span>exact_al<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	exact_number<span style='color:#800080; '>&lt;</span>T<span style='color:#800080; '>></span> sum_bl_br <span style='color:#808030; '>(</span>exact_bl<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>

	<span style='color:#696969; '>/*</span>
<span style='color:#696969; '>	    Variable explanation</span>
<span style='color:#696969; '>	    sum_al_ar = al + ar</span>
<span style='color:#696969; '>	    sum_bl_br = bl + br</span>
<span style='color:#696969; '>	*/</span>
	sum_al_ar<span style='color:#808030; '>.</span>add_vector<span style='color:#808030; '>(</span>exact_ar<span style='color:#808030; '>,</span> base <span style='color:#808030; '>-</span> <span style='color:#008c00; '>1</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	sum_bl_br<span style='color:#808030; '>.</span>add_vector<span style='color:#808030; '>(</span>exact_br<span style='color:#808030; '>,</span> base <span style='color:#808030; '>-</span> <span style='color:#008c00; '>1</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>

	exact_al<span style='color:#808030; '>.</span>karatsuba_multiplication<span style='color:#808030; '>(</span>exact_bl<span style='color:#808030; '>,</span> base<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	sum_al_ar<span style='color:#808030; '>.</span>karatsuba_multiplication<span style='color:#808030; '>(</span>sum_bl_br<span style='color:#808030; '>,</span> base<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	exact_ar<span style='color:#808030; '>.</span>karatsuba_multiplication<span style='color:#808030; '>(</span>exact_br<span style='color:#808030; '>,</span> base<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#696969; '>/*</span>
<span style='color:#696969; '>	    Variable explanation</span>
<span style='color:#696969; '>	    exact_al = al * bl</span>
<span style='color:#696969; '>	    sum_al_ar = (al + ar) * (bl + br)</span>
<span style='color:#696969; '>	    exact_ar = ar * br</span>
<span style='color:#696969; '>	*/</span>

	sum_al_ar<span style='color:#808030; '>.</span>subtract_vector<span style='color:#808030; '>(</span>exact_al<span style='color:#808030; '>,</span> base <span style='color:#808030; '>-</span> <span style='color:#008c00; '>1</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	sum_al_ar<span style='color:#808030; '>.</span>subtract_vector<span style='color:#808030; '>(</span>exact_ar<span style='color:#808030; '>,</span> base <span style='color:#808030; '>-</span> <span style='color:#008c00; '>1</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#696969; '>/*</span>
<span style='color:#696969; '>	    sum_al_ar = al * br + ar * bl</span>
<span style='color:#696969; '>	*/</span>

	<span style='color:#696969; '>// multiply exact_al by base^(2 * right_half_length)</span>
	exact_al<span style='color:#808030; '>.</span>exponent <span style='color:#808030; '>+</span><span style='color:#808030; '>=</span> <span style='color:#008c00; '>2</span> <span style='color:#808030; '>*</span> right_half_length<span style='color:#800080; '>;</span>

	<span style='color:#696969; '>// multiply sum_al_ar by base^(right_half_length)</span>
	sum_al_ar<span style='color:#808030; '>.</span>exponent <span style='color:#808030; '>+</span><span style='color:#808030; '>=</span> right_half_length<span style='color:#800080; '>;</span>

	exact_al<span style='color:#808030; '>.</span>add_vector<span style='color:#808030; '>(</span>sum_al_ar<span style='color:#808030; '>,</span> base <span style='color:#808030; '>-</span> <span style='color:#008c00; '>1</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	exact_al<span style='color:#808030; '>.</span>add_vector<span style='color:#808030; '>(</span>exact_ar<span style='color:#808030; '>,</span> base <span style='color:#808030; '>-</span> <span style='color:#008c00; '>1</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>

	<span style='color:#808030; '>*</span><span style='color:#800000; font-weight:bold; '>this</span> <span style='color:#808030; '>=</span> exact_al<span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>exponent <span style='color:#808030; '>+</span><span style='color:#808030; '>=</span> <span style='color:#808030; '>-</span><span style='color:#808030; '>(</span>a_size <span style='color:#808030; '>+</span> b_size<span style='color:#808030; '>)</span> <span style='color:#808030; '>+</span> <span style='color:#808030; '>(</span>a_exponent <span style='color:#808030; '>+</span> b_exponent<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>positive <span style='color:#808030; '>=</span> <span style='color:#808030; '>(</span>a_sign <span style='color:#808030; '>=</span><span style='color:#808030; '>=</span> b_sign<span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>
	<span style='color:#800000; font-weight:bold; '>this</span><span style='color:#808030; '>-</span><span style='color:#808030; '>></span>normalize<span style='color:#808030; '>(</span><span style='color:#808030; '>)</span><span style='color:#800080; '>;</span>

<span style='color:#800080; '>}</span>
</pre>
<!--Created using ToHtml.com on 2020-08-29 07:08:54 UTC -->

Initial commit regarding Karatsuba algorithm:   [https://github.com/BoostGSoC20/Real/commit/83e43dede5e2d101240777fc1eb0dd14a64f3eba](https://github.com/BoostGSoC20/Real/commit/83e43dede5e2d101240777fc1eb0dd14a64f3eba)  

### Tests and Documentation
All required tests and documentation were made in parallel with the code implementation. 
Following is overview of how test for pi has been implemented :  

```cpp
SECTION("TRIGONOMETRIC PROPERTIES") {

	SECTION("sin(pi) == 0") {

		real sin_pi = real::sin(pi);
		boost::real::exact_number zero("0");

		auto sin_pi_iterator = sin_pi.get_real_itr().cbegin();
		++sin_pi_iterator;

		CHECK(sin_pi_iterator.get_interval().lower_bound <= sin_pi_iterator.get_interval().upper_bound);
		CHECK(!sin_pi_iterator.get_interval().lower_bound.positive);
		CHECK(sin_pi_iterator.get_interval().upper_bound.positive);
		++sin_pi_iterator;

		CHECK(sin_pi_iterator.get_interval().lower_bound <= sin_pi_iterator.get_interval().upper_bound);
		CHECK(!sin_pi_iterator.get_interval().lower_bound.positive);
		CHECK(sin_pi_iterator.get_interval().upper_bound.positive);
		++sin_pi_iterator;

		CHECK(sin_pi_iterator.get_interval().lower_bound <= sin_pi_iterator.get_interval().upper_bound);
		CHECK(!sin_pi_iterator.get_interval().lower_bound.positive);
		CHECK(sin_pi_iterator.get_interval().upper_bound.positive);
		++sin_pi_iterator;

		CHECK(sin_pi_iterator.get_interval().lower_bound <= sin_pi_iterator.get_interval().upper_bound);
		CHECK(!sin_pi_iterator.get_interval().lower_bound.positive);
		CHECK(sin_pi_iterator.get_interval().upper_bound.positive);
		++sin_pi_iterator;
	}

}

SECTION("e^pi > pi^e") {

	real one("1");
	real e("2.718281828459045235360287471352662497757");
	real e_raised_to_pi = real::power(e, pi);
	real pi_raised_to_e = real::power(pi, e);

	auto itr1 = e_raised_to_pi.get_real_itr().cbegin();
	auto itr2 = pi_raised_to_e.get_real_itr().cbegin();

	++itr1;
	++itr2;

	CHECK(itr1.get_interval().lower_bound >= itr2.get_interval().upper_bound);

	++itr1;
	++itr2;

	CHECK(itr1.get_interval().lower_bound >= itr2.get_interval().upper_bound);

	++itr1;
	++itr2;

	CHECK(itr1.get_interval().lower_bound >= itr2.get_interval().upper_bound);

}
```

## Commit History  
[Here](https://github.com/BoostGSoC20/Real/commits?author=kishanshukla-2307) is the list of all the commits that were merged in Boost.Real master during or before the start of GSoC 2020 coding period.  

## Areas for future improvements
There are few things which can be optimized in the library.  
- The implementation of chudnovsky algorithm for nth digit of pi can be more optimal as it performs redundant calculations in evaluating each new digit of pi. If the maximum number of digits of pi user is going to use is known prior, then the redundent calculations could be avoided but then all the calculations would be performed at higher precision (depending on the number of digits of pi user needs) therefore calculation of each digit becomes slower but each new digit is obtained quickly using the previous calculation. Something more smarter or the idea I suggested can be used to optimize calculation of pi digits.
- Similarly, evaluation of taylor series involves redundant calculation and that is why all the trigonometric functions, logarithm, fractional power, etc are very slow.
- The Newton-Raphson division algorithm uses an initial guess evaluation of which involves division itself. Due to this circular dependancy I implemented an approximate division method to calculate the initial guess roughly, as the intial guess was crucial in ensuring faster convergence of the newton-raphson algorithm. This initial guess can be replaced with something that doesn't involve division and hence the use of the approximate division algorithm can be avoided.

## Remarks
It was an amazing three months where I got to work with experienced mentors __Laouen Belloli__ and __Damian Vicino__ and also a very talented colleague __Vikram Chundawat__. It was a great learning experience in terms of writing clean and efficient cpp codes. All those critical code reviews by the mentors improved us a lot. At last, I want to thank __Google__ and __Boost c++__ for providing me the opportunity to work with the Boost community. :)
