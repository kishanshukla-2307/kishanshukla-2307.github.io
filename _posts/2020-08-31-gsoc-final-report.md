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


<pre style='color:#000000;background:#D3D3D3;'><span style='color:#800000; font-weight:bold; '>case </span><span style='color:#7d0045; '>OPERATION</span><span style='color:#800080; '>::</span><span style='color:#7d0045; '>INTEGER_POWER</span><span style='color:#e34adc; '>:</span> <span style='color:#800080; '>{</span>
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


```cpp
template <typename T = int>
T pi_nth_digit(unsigned int n) {
	// Chudnovsky Algorithm
	// pi = C * ( sum_from_k=0_to_k=x (Mk * Lk / Xk) )^(-1) 
	// increasing x you get more precise pi

	static const boost::real::real_explicit<T> real_k("6");
	static const boost::real::real_explicit<T> real_m("1");
	static const boost::real::real_explicit<T> real_l("13591409");
	static const boost::real::real_explicit<T> real_l0("545140134");
	static const boost::real::real_explicit<T> real_x("1");
	static const boost::real::real_explicit<T> real_x0("-262537412640768000");
	static const boost::real::real_explicit<T> real_s("13591409");

	static exact_number<T> L0 = real_l0.get_exact_number();
	static exact_number<T> X0 = real_x0.get_exact_number();

	bool nth_digit_found = false;
	bool first_iteration_over = false;

	exact_number<T> iteration_number = one;
	exact_number<T> prev_pi;
	exact_number<T> pi;
	exact_number<T> error;
	const exact_number<T> max_error(std::vector<T> {1}, -(n + 1), true);

	do {  
	    exact_number<T> temp = K * K * K - _16 * K;
	    temp.divide_vector(iteration_number * iteration_number * iteration_number, n + 1, true);
	    M *= temp;
	    X *= X0;
	    L += L0;
	    K += _12;

	    temp = M * L;
	    temp.divide_vector(X, n + 1, false);
	    S += temp;

	    temp = C;
	    temp.divide_vector(S, n + 1, false);

	    pi = temp;
	    if (!first_iteration_over) {
		prev_pi = pi;
		first_iteration_over = true;
		iteration_number += one;
	    } else {
		error = pi - prev_pi;
		error.positive = true;

		if (error < max_error) {
		    nth_digit_found = true;
		}
		iteration_number += one;
		prev_pi = pi;
	    }

	} while (!nth_digit_found);

	return pi[n];
}
```

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


```cpp
template <typename T = int>
void karatsuba_multiplication (
    exact_number<T> &other, 
    const T base = (std::numeric_limits<T>::max() / 4) * 2
) {

	// this --- a, other --- b
	const int a_size = this->digits.size();
	const int b_size = other.digits.size();
	const int a_exponent = this->exponent;
	const int b_exponent = other.exponent;
	const bool a_sign = this->positive;
	const bool b_sign = other.positive;

	const int max_length = std::max(a_size, b_size);

	if (max_length <= KARATSUBA_BASE_CASE_THRESHOLD || std::abs(a_size - b_size) > std::min(a_size, b_size)) {
	    this->standard_multiplication(other, base);
	    return;
	}

	// appending zeroes in front to make sizes of a & b equal
	int a_pref_zeroes = 0, b_pref_zeroes = 0;
	if (a_size < max_length) {
	    a_pref_zeroes = max_length - a_size;
	} else if (b_size < max_length) {
	    b_pref_zeroes = max_length - b_size;
	} 

	const int left_half_length = max_length / 2;
	const int right_half_length = max_length - left_half_length;

	/*
		Variable Explanation
	    a is vector representation of "this", b is vector representation of "other"
	    a = exact_al * base^(right_half_length) + exact_ar
	    b = exact_bl * base^(right_half_length) + exact_br
	*/
	
	// This is where we perform optimization (not appending zeroes in vector)
	if (a_pref_zeroes > 0) {
	    if (a_pref_zeroes >= left_half_length) {
		exact_al = exact_number(std::vector<T> (), true);
		exact_ar = exact_number(this->digits, true);
	    } else {
		exact_al = exact_number(std::vector<T> (this->digits.begin(), this->digits.begin() + left_half_length - a_pref_zeroes), true);
		exact_ar = exact_number(std::vector<T> (this->digits.begin() + left_half_length - a_pref_zeroes, this->digits.end()), true);
	    }
	    exact_bl = exact_number(std::vector<T> (other.digits.begin(), other.digits.begin() + left_half_length), true);
	    exact_br = exact_number(std::vector<T> (other.digits.begin() + left_half_length, other.digits.end()), true);
	} else if (b_pref_zeroes > 0) {
	    if (b_pref_zeroes >= left_half_length) {
		exact_bl = exact_number(std::vector<T> (), true);
		exact_br = exact_number(other.digits, true);
	    } else {
		exact_bl = exact_number(std::vector<T> (other.digits.begin(), other.digits.begin() + left_half_length - b_pref_zeroes), true);
		exact_br = exact_number(std::vector<T> (other.digits.begin() + left_half_length - b_pref_zeroes, other.digits.end()), true);
	    }
	    exact_al = exact_number(std::vector<T> (this->digits.begin(), this->digits.begin() + left_half_length), true);
	    exact_ar = exact_number(std::vector<T> (this->digits.begin() + left_half_length, this->digits.end()), true);
	} else {
	    exact_al = exact_number(std::vector<T> (this->digits.begin(), this->digits.begin() + left_half_length), true);
	    exact_ar = exact_number(std::vector<T> (this->digits.begin() + left_half_length, this->digits.end()), true);
	    exact_bl = exact_number(std::vector<T> (other.digits.begin(), other.digits.begin() + left_half_length), true);
	    exact_br = exact_number(std::vector<T> (other.digits.begin() + left_half_length, other.digits.end()), true);
	}

	exact_number<T> sum_al_ar (exact_al);
	exact_number<T> sum_bl_br (exact_bl);

	/*
	    Variable explanation
	    sum_al_ar = al + ar
	    sum_bl_br = bl + br
	*/
	sum_al_ar.add_vector(exact_ar, base - 1);
	sum_bl_br.add_vector(exact_br, base - 1);

	exact_al.karatsuba_multiplication(exact_bl, base);
	sum_al_ar.karatsuba_multiplication(sum_bl_br, base);
	exact_ar.karatsuba_multiplication(exact_br, base);
	/*
	    Variable explanation
	    exact_al = al * bl
	    sum_al_ar = (al + ar) * (bl + br)
	    exact_ar = ar * br
	*/

	sum_al_ar.subtract_vector(exact_al, base - 1);
	sum_al_ar.subtract_vector(exact_ar, base - 1);
	/*
	    sum_al_ar = al * br + ar * bl
	*/

	// multiply exact_al by base^(2 * right_half_length)
	exact_al.exponent += 2 * right_half_length;

	// multiply sum_al_ar by base^(right_half_length)
	sum_al_ar.exponent += right_half_length;

	exact_al.add_vector(sum_al_ar, base - 1);
	exact_al.add_vector(exact_ar, base - 1);

	*this = exact_al;
	this->exponent += -(a_size + b_size) + (a_exponent + b_exponent);
	this->positive = (a_sign == b_sign);
	this->normalize();

}
```
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
