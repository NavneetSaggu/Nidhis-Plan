# C++ STL

## Functions Supported By Various Data Structures

```cpp
	priority_queue
		push(), pop(), top()

	queue
		push(), pop(), front(), back(), empty(), size()

	set
		find(), insert(), erase()

	map
		[], find(key), insert(), begin(), end(), count(key), erase(), clear()
		NOTES
			You can operate directly on empty maps
				map<int, vector<int>> m; m[1].push_back(2); is perfectly fine.
				unordered_map<string, unordered_map<string, double>> pathVal; pathVal["Hello"]["World"] = 1.0; is perfectly fine
			You can check contains with count
				if(pathVal.count("Hello"))
			        cout<<"Has hello";
			  if(!pathVal.count("Lol"))
			        cout<<"No has Lol";

	unordered_map
		[], find(key), insert(), begin(), end(), count(key)

	vector
		[]
		push_back()
		pop_back()
		erase()
				vector_name.erase(position);    
				vector_name.erase(starting_position, ending_position); 
		resize()
		(copies values from one to another), insert()
			NOTES
				You can initialize things like vector<pair<int, int>> l = { { INT_MAX, 0 }, { INT_MAX, 0 } };
		distance(it1, it2) will return an integer denoting the distance between any two iterators

	string
		[]
		length()
		rfind()
				rfind searches for the last (rightmost) occurrence of a specified substring within the given string.
				It starts searching from the end (i.e., the last character) of the string and moves towards the beginning.
				If it finds the substring, it returns the index (position) of the first character of the last occurrence of the substring within the string.
				If the substring is not found, it returns std::string::npos.
				std::string test = "0123123";
				size_t match1 = test.rfind("123");    // returns 4 (rightmost match)
				size_t match2 = test.rfind("123", 2); // returns 1 (skipped over later match)
				size_t match3 = test.rfind("123", 0); // returns std::string::npos (i.e. not found)
		find()
				find searches for the first occurrence of a specified substring within the given string.
				It starts searching from the beginning (i.e., the first character) of the string and continues towards the end.
				If it finds the substring, it returns the index (position) of the first character of the found substring within the string.
				If the substring is not found, it returns std::string::npos.
				size_t pos = mainString.find(substring);
		    while (pos != std::string::npos) {
		        std::cout << "Found at position: " << pos << std::endl;
		        pos = mainString.find(substring, pos + 1);
		    }		

	pair
		NOTES
			You can insertly directly by using {el1, el2};
				queue<pair<string, double>> Q; Q.push({"Hello", 0.1}); is perfectly fine
			You can unpack using this
				pair<int, int> p(2,3);
				auto[x,y] = p;
			You can initialize as 
				pair<int, int> p = {2,3};

	priority_queue
		NOTES
			priority_queue <int> makes max heap
			priority_queue <int, vector<int>, greater<int>> makes min heap

	for
		for(int i :{1,2,3}) {
			// something with i
		}
```

## Other Pointers

```java
BEAUTIFUL SOLUTIONS
	1. https://leetcode.com/problems/search-in-rotated-sorted-array-ii/discuss/28218/My-8ms-C%2B%2B-solution-(o(logn)-on-average-o(n)-worst-case)
	2. https://leetcode.com/problems/min-stack/discuss/49016/C%2B%2B-using-two-stacks-quite-short-and-easy-to-understand
	3. https://leetcode.com/problems/contains-duplicate-iii/
	4. https://leetcode.com/problems/majority-element-ii/
	5. https://leetcode.com/problems/h-index/

PREP DEBT
	1. KMP Algorithm
	2. Morris Traversal
	3. Manachers algorithm
	4. https://leetcode.com/problems/single-number-ii/discuss/43295/Detailed-explanation-and-generalization-of-the-bitwise-operation-method-for-single-numbers
	5. Catalan Number

LEARNINGS
	0. C++ POINTERS ARE NOT PASSED BY REFERENCE
	1. swap(x, y) STL to swap any two things in place (strings, vectors, objects); swap(arr[x], arr[y]) will also work to swap elements
	2. string word = "abcde"; word.substr(2, 2) is "cd" (initial pos, size from pos) && word.substr(2) is "cde" (initial pos to the end)
	3. string word1 = word; creates new string word1 and not an alias
	// 4. dynamically allocating arrays for passing
	// 	int m = 5, n = 4;
 //    	bool **visited = new bool*[m];
 //        for(int i=0; i<m; i++){
 //            visited[i] = new bool[n];
 //            for(int j=0; j<n; j++){
 //                visited[i][j] = 7;
 //            }
 //        }
    5. for custom sorting (non-decreasing
    	sort(mMyClassVector.begin(), mMyClassVector.end(), 
	    	[](const MyClass & a, const MyClass & b) -> bool { return a.mProperty < b.mProperty; });
    5.a. for custom stable sorting, use
    	stable_sort(map.begin(), map.end(), [](const pair<int,int> &x, const pair<int,int> &y)->bool{return x.second > y.second;});
    		this will do a stable decreasing sort of pairs on their second value
    6. upper_bound gives first element > val in non-decreasing array
    	lower_bound gives first element >= val in non-decreasing array
    	[5, <- l_b of 4
    	 6,	<- l_b of 6
    	 6,
    	 7, <- u_b of 6
    	 8]
   	7. insert(vector<T>::iterator it, T value) inserts value at it position, shifting elements from it -> vec.end() right by one
   		it returns iterator to the newly inserted element; ammortized complexity is O(1);
   		We can also do a insert(col.end(), row.begin(), row.end()); to insert entire row array at cols end
   	8. non-decreasing (el1 <= el2)
   		non-increasing (el1 >= el2)
   	// 9. to sort in non-increasing order use std::greater<int>() operator (3rd parameter)
   	// 	can also be used for l_b and u_b in such cases (4th parameter)
   	// 	upper_bound gives first element < val
   	// 	lower_bound gives first element <= val
   	// 	[8, <- l_b of 9
    // 	 7,	
    // 	 6, <- l_b of 6
    // 	 6,
    // 	 5, <- u_b of 6
    // 	 4]
    10. use *  to dereference iterators
    11. distance(arr.begin(), it) to find distance between two iterators
    12. find(arr.begin(), arr.end(), val) to see if val exists in sorted array
    13. arr.begin() -> iterator to first element (exists) (increment moves it towards back)
    	arr.end() -> iterator to the element beyond the last element (doesnt exist) (increment moves it towards back)
    	arr.rbegin() -> iterator to the last element (exists) (increment moves it towards front)
    14. initialize vectors like return vector <int>(2, -1); params are (num of values, value); can also give elements like vector<int>{1,2,3,4,5,6};
    15. ASCII of 
    	'0'  is 48
    	'A'  is 65
    	'a'  is  97
    16. Converting vector of chars into string String str(vec.begin(), vec,end());
    17. EVERYTHING in C++ (even vectors and other complex data structures) are passed by value
    18. Maps and Sets in STL are sorted, map insertion is log(n), but if you insert with hint pointing to position where element will be inserted, its O(1) (insert(hint, pair), similar to vector)
    19. queue/vector.front() and queue/vector.back() give the first and the last elements of a queue/vector
    20. Iterators for string and vector support relational operators which include >, >=, <, <=
    21. Sort and Find support use of lambdas
    	- sort
    		sort(arr.begin(), arr.end(), [] (const pair<int,int> x, const pair<int,int> y) -> bool {return x.second < y.second;});
    	- lower_bound / upper_bound
    		lower_bound(arr.begin(), arr.end(), pair<int, int>(3,3),  [] (const pair<int,int> x, const pair<int,int> y) -> bool {return x.second > y.second;})
    		Finds the first element ≤ target in the decreasing array, so it's all focussed on the first element, on the next one in the vector going from left to right
    22. int goes from INT_MIN to INT_MAX
    	long long int goes form LLONG_MIN to LLONG_MAX
    23. unordered_set<> complexity of insertion(), find(), erase() is all O(1) average, but worst case is O(n). Uses hash table internally
    24. list<> is implemented as a double linked list, I think. So insertions and deletions at any and all positions are O(1).
    	push_front(), push_back(), pop_front(), pop_back(), insert() are all supported and are similar to those seen in vector and dequeue. (dequeue also supports random access op i.e. []) 
    	Also supports front() and back();
    	remove() takes value as input -> Complexity is linear in container size (comparisons)
    	erase() takes iterator as input -> Compexity is linear in the number of elements erased
    	IMPORTANT 		vector iterators will change after you push in new values, probably because broken contiguous nature of it, whereas 
    				list iterators will remain the same, because of the entirely non-contiguous nature of it.
    25. minmax_element(nums.begin(), nums.end()), returns a pair<int, int> of minimum val in range, maximum val in range
    26 string constructor, (size, char to be repeated)
    27. stoi(<string>) gives integer, to_string(<integer>) gives string [to string tostring]
    28. abs() for int, fabs() for float, labs() for long
    29. int64_t is a signed integer type with exactly 64 bits. long long int is a signed integer type with at least 32 bits.
    30. fill( vector<>::it1, vector<>::it2 , val) fills container from it1 to it2 with value val
    31. A DAG will always have atleast one node with indegree 0. 
    	BFS algo for topsort is built on this (https://www.hackerearth.com/practice/algorithms/graphs/topological-sort/tutorial) 
    	(same for outdegree https://math.stackexchange.com/questions/3013656/having-no-directed-cycles-guarantees-a-vertex-of-zero-outdegree). 
    32. For min_heap
			auto comp = [](ListNode *a, ListNode *b) -> bool { return a->val > b->val; };
			priority_queue<ListNode *, vector<ListNode *>, decltype(comp)> pq(comp);

			auto comp = [](const pair<int, int> a, const pair<int, int> b) -> bool {return a.first > b.first;};
			priority_queue<pair<int, int>, vector<pair<int, int>>, decltype(comp)> pq(comp);

			Beautiful explanations https://www.hackerearth.com/practice/notes/heaps-and-priority-queues/
	33. The rbegin() is a function in C++ STL. It returns a reverse iterator which points to the last element of the map. The reverse iterator iterates in reverse order and incrementing it means moving towards beginning of map. i.e. for (auto i = mp.rbegin(); i != mp.rend(); ++i)
	41. Iterators
		to get iterator of an element in 
	34. DFS using stack
		TN* node = root;
		while(node != NULL || !stack.empty()) {
			while(node) {
				stack.push(node);
				node = node->left;
			}
			node = stack.top();
			stack.pop();
			process(node);
			node = node->right;
		}
	35. Topological Sort
		DFS:
			1. Pick unvisited node
			2. Do DFS from this node, visiting only unvisited nodes
			3. On recursive call back, add current  node to top order in reverse (at front)
			4. Repeat for all unvisited nodes
		BFS (Kahns):
			1. (https://www.youtube.com/watch?v=cIBFEhD77b4&ab_channel=WilliamFiset)
	36. To the LSB that is set to 1, i.e. 
			0000 0000 0000 0000 0000 0000 0010 0000 = 32 from 
			0000 0000 0000 0000 0000 0000 0110 0000 = 96
			x = x & (-x);
	37. BIT ENCODING STUFF
		 	1s compliment is just switching bits			2s compliment is 1s compliment + 1
			All -ve numbers are 2s compliment, their MSB is 1 (LSB is rightmost, MSB is leftmost)
		FOR INTUITION
			Say we have an 8-bit encoding
				POSITIVE RANGE
					0000 0000 : 0
					0000 0001 : 1
					.
					.
					0111 1110 : 126
					0111 1111 : 127
				
				NEGATIVE RANGE
					1111 1111 : -1
					1111 1110 : -2
					1111 1101 : -3
					.
					(goes backwards)
					.
					1000 0001 : -127
					1000 0000 : -128 (the one extra magnitude, compared to +ves)
	38. For O(1) space complexity requirements, ask if output array counts as extra space
	39. A collection of bit manipulation problems 
		https://leetcode.com/problems/sum-of-two-integers/discuss/84278/A-summary%3A-how-to-use-bit-manipulation-to-solve-problems-easily-and-efficiently
		https://leetcode.com/problems/sum-of-two-integers/discuss/132479/Simple-explanation-on-how-to-arrive-at-the-solution	
	40. Best way to calculate nCr to minimize overflows
		for(i = 1; i <= r; i++) {
	        ans *= n - r + i;
	        ans /= i;
	    }
	42. Map operations
		Using erase() : erase() is used to erase the pair in map mentioned in argument, either its position, its value or a range of number.
			erase(key) : Erases the key-value pair using key mentioned in its argument. reorders the map after deletion. It returns the number of entries deleted. If non-existing keys is deleted, 0 is returned.
			Time complexity : log(n) (n is size of map)
			erase(iter) : Erases the pair at the position pointed by the iterator mentioned in its argument.
			Time complexity : log(n) (n is size of map)
			erase(strt_iter, end_iter) : Erases the range of pairs starting from “strt_iter” to the “end_iter”.
			Time complexity : O(k) (k is size of map)
	43. Make an empty constructor every time you try to make a custom class to make objects to put in your containers
	44. Hashes take more memory than other indexed data structures (https://stackoverflow.com/questions/22502412/why-does-a-hash-table-take-up-more-memory-than-other-data-structures)
		we add additional buckets that can remain empty and the overhead associated with these buckets is the additional memory taken up by a hash table?
		that os the primary reason for the extra usage, add to that the extra memory being used by the collision resolution method, such as linked lists.
	45. Use space seperated istringstreams and ostringstreams (https://leetcode.com/problems/serialize-and-deserialize-binary-tree/discuss/74259/Recursive-preorder-Python-and-C%2B%2B-O(n))
	46. Optimal Substructure: If optimal solution to a problem, S, of size n can be calculated by JUST looking at optimal solution of a subproblem, s, with size < n and NOT ALL solutions to subproblem, AND it will also result in an optimal solution for problem S, then this problem S is considered to have optimal substructure.
	47. strcmp(left, right) basically does a left - right. For eg. strcmp("zfz","gfg") is > 0, 19
		"aa" < "bb" is true, you can compare strings 
	48. Kadane for empty subarray
			int sum = 0;
		   	int MAX = INT_MIN;
		   	for(int x : a) {
		       	sum += x;
		       	if(sum < 0)
		            sum = 0;
		        MAX = max(MAX, sum);
		Kadane for non-emoty subarray
			int sum = 0;
		   	int MAX = INT_MIN;
		   	for(int x : a) {
		       	sum += x;
		       	MAX = max(MAX, sum);
		       	if(sum < 0)
		            sum = 0;
	49. LLONG_MIN, LLONG_MAX range for long long
	50. iota(parent.begin(), parent.end(), 0); will fill the space in the given iterators for the container with values from 0 anf increment by 1, so 0,1,2,3,4...
	52. Tuples
			    // Declaring tuple
			    tuple <char, int, float> geek;
			  
			    // Assigning values to tuple using make_tuple()
			    geek = make_tuple('a', 10, 15.5);
			  
			    // getting values of tuples
			    cout << get<0>(geek) << " " << get<1>(geek);
			  
			    // Use of get() to change values of tuple
			    get<0>(geek) = 'b';
			    get<2>(geek) =  20.5;
	53. To get nth largest element in a collection, simply use nth_element
				int findKthLargest(vector<int>& nums, int k) {
					nth_element(nums.begin(),nums.begin()+nums.size()-k,nums.end());
					return nums[nums.size()-k];
				}
		        

APPROACH ADVICE
	1. When everything seems fugazi, try to define your data structures and variables; write down properties that your data structures and variables will obey, under what conditions, and at what time.
	2. Learn to write good, stripped down, bare minimum psuedocode, it is a very very valuable skill.
	3. Dont go looking for minimalistic/elegant solutions straightaway; first formulate a solution that covers are base cases, elaborate it to cover edge cases. Elegant solutions are a bonus that come after the problem has been solved, not a necessity.

GRAPH THINGS
	1. DFS traversal will form a tree
	2. A bipartite graph will have chromatic number of 2 
	3. There can be max n_color_1*n_color_2 edges in a bipartite graph
	4. There will be N-1 edges in an MST
	5. Finding chromatic number of a graph is an np complete problem
	6. A Back Edge is an edge that connects a vertex to a vertex that was discovered before its parent
		There is a certain set of back edges for each DFS traversal. These can change if the order of traversal changes.
		Order of traversal, however, does not change; as our adjacency list and our algorithm remains the same.
		Presence of back edge indicates a cycle in directed graph
		By their very definition, back edges can never be bridges
	7. A Cross Edge is an edge which connects two node such that they do not have any ancestor and a descendant relationship between them.
	8. If edge u->v is a bridge, there is no back edge from v or the subgraph of v, to u or any ancestor of u.
		This is the concept leveraged by Tarjans algorithm to find bridges in O(V+E) timne

MATH THINGS
	1. gcd(a, b) { return  b == 0 ? a : gcd(b, a % b); }
	2. Modular Inverse
		The multiplicative inverse of “A modulo M” exists if and only if A and M are relatively prime (i.e. if gcd(A, M) = 1)
		METHOD 1
			For any two integers ‘a’ and ‘b’, their gcd, can be expressed as a linear combination of 'a' and 'b' (here x and y are integers)
			ax + by = gcd(a, b)
			
			We use this idea to find the multiplicative inverse of ‘A’ under ‘M’; we put b = M in the above formula. Since we know that A and M are relatively prime, we can put the value of gcd as 1.
			Ax + My = 1
			
			If we take modulo M on both sides, we get
			Ax  ≅ 1 (mod M)

			Thus the x coefficient in this equation will be the modular inverse of A
			x and y can be calculated using the Extended Euclid Algorithm. This just works backwards from the Euclidean Algorithm for GCD, as given in  this video https://www.youtube.com/watch?v=hB34-GSDT3k&ab_channel=GVSUmath

			// Returns modulo inverse of a with respect
			// to m using extended Euclid Algorithm
			// Assumption: a and m are coprimes, i.e.,
			// gcd(A, M) = 1
			int modInverse(int A, int M)
			{
			    int m0 = M;
			    int y = 0, x = 1;
			 
			    if (M == 1)
			        return 0;
			 
			    while (A > 1) {
			        // q is quotient
			        int q = A / M;
			        int t = M;
			 
			        // m is remainder now, process same as
			        // Euclid's algo
			        M = A % M, A = t;
			        t = y;
			 
			        // Update y and x
			        y = x - q * y;
			        x = t;
			    }
			 
			    // Make x positive
			    if (x < 0)
			        x += m0;
			 
			    return x;
			}

		METHOD 2
			This algorithm is based on Fermats little theorem
				Fermat’s little theorem states that if p is a prime number, then for any integer a, the number a^p – a is an integer multiple of p. 
				Here p is a prime number 
				a^p ≡ a (mod p)      <== MAIN RESULT FROM FERMATS THAT WE USE
				a^(p-1) ≡ 1 (mod p)

				This gives us 
					a^(m-2) ≡ a^(-1) (mod m)

			// Function to find modular inverse of a under modulo M
			// Assumption: M is prime
			void modInverse(int A, int M)
			{
			    int g = gcd(A, M);
			    if (g != 1)
			        cout << "Inverse doesn't exist";
			    else {
			        // If a and m are relatively prime, then modulo
			        // inverse is a^(m-2) mode m
			        cout << "Modular multiplicative inverse is "
			             << power(A, M - 2, M);
			    }
			}
			 
			// To compute x^y under modulo m
			int power(int x, unsigned int y, unsigned int M)
			{
			    if (y == 0)
			        return 1;
			 
			    int p = power(x, y / 2, M) % M;
			    p = (p * p) % M;
			 
			    return (y % 2 == 0) ? p : (x * p) % M;
			}
			 
			// Function to return gcd of a and b
			int gcd(int a, int b)
			{
			    if (a == 0)
			        return b;
			    return gcd(b % a, a);
			}
```