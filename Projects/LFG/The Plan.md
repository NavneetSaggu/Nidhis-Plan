# Low Level Design

- [ ] Java Fundamentals
	- [ ] Go through the complete refresher content here https://drive.google.com/drive/folders/1V-HTZxAoqRly88oWgtkR0VdJ0aUuxXvY
	- [ ] [Optional] ⁠Shreyanshs concept and coding videos if you want a more thorough revision. The complete playlist is paid content but it’s worth it https://youtube.com/playlist?list=PL6W8uoQQ2c63f469AyV78np0rbxRFppkx&si=dVdpz-BD56Tm6DSo
	- [ ] Cover everything in [[_Java]]. Noting the important ones here
		- [ ] [[Core Java]]
		- [ ] [[Access Modifiers]]
		- [ ] [[JIT Compiler]]
		- [ ] [[Functional Interface]]
		- [ ] [[Collections]] and [[Java Collections]]
		- [ ] [[_Garbage Collection]]
		- [ ] [[_Thread Lifecycle]]
		- [ ] [[ThreadLocal.pdf]]
		- [ ] [[_Thread Pools on Java Servers]]
		- [ ] [[Singletons]]
		- [ ] [[_Java Concurrency]]
		- [ ] [[_Java Memory]]
		- [ ] [[ProducerConsumer.pdf]]
		- [ ] [[_Hibernate]]
		- [ ] [[Maven]]
		- [ ] [[Packagings]]
- [ ] Practice LLD questions from here. Focus on questions that have multi threading ke concepts. Try and solve things yourself first and then comparing your solutions https://www.lldcoding.com/

# High Level Design

- [ ] Complete Chapters 1 through 6 from Kleppman's notes [[_Designing Data Intensive Systems]]
	- [ ] [[Chapter 1 - Reliable, Scalable & Maintainable Applications]]
	- [ ] [[Chapter 2 - Data Models & Query Languages]]
	- [ ] [[Chapter 3 - Storage & Retrieval]]
	- [ ] [[Chapter 4 - Encoding & Evolution]]
	- [ ] [[Chapter 5 - Replication]]
	- [ ] [[Chapter 6 - Partitioning]]
- [ ] Complete Chapters 7 through 11 from [[Martin Kleppmann - Designing Data-Intensive Applications. The Big Ideas Behind Reliable, Scalable and Maintainable Systems-O’Reilly (2017).pdf]]
	- [ ] Chapter 7
	- [ ] Chapter 8
	- [ ] Chapter 9
	- [ ] Chapter 10
	- [ ] Chapter 11
- [ ] Read through Alex Xu. No need to memorise anything, just revise the concepts once.
	- [ ] VV Important, read first chapter of Alex Xu to help with back-of-the-envelope stuff
- [ ] DBMS Stuff
- [ ] Listen to ALL the hello interview problems, memorise the designs themselves, all the tradeoffs discussed and all the technologies discussed.

# Data Structures And Algorithms

- [ ] We'll be doing all our coding in Java now, start by revising Java collections from
	- [ ] [[Java Collections]]
	- [ ] [[Collections]]
- [ ] Solve Strivers A2Z sheet https://takeuforward.org/dsa/strivers-a2z-sheet-learn-dsa-a-to-z It might seem excessive but you’ll be a DSA god by the end of it
- [ ] A month before interviews, buy LC premium and practice company specific questions for companies you’re prepping for.

Most important, do DSA the right way. Always practice topic wise, one week per topic. Start with reading theory, learning basic algorithms, and reading through basic questions. Only then begin with questions from the sheet. Set a 10 min time limit for your solution. If you haven’t thought of a viable approach in 10 mins, read the solution and move on.

⁠If you have thought of a viable approach in 10 mins, take no longer than 15 mins to code it. Speed is paramount, you need to be able to solve and solve fast.

⁠Towards the end of your prep, whenever you’re solving questions, solve them in a mock interview format. Practice how you’ll walk the interviewer through your solution and what pseudocode youll write to explain your thinking to them and get buy in.

# Interview Experiences

Uber ka ek aur DSA type question tha in another round
Meeting scheduling, but a little different
You have a set of rooms given, each room has a roomId
Your meeting scheduler will have a function called schedule() which will take start time and end time for your meeting, block a suitable room for that time, and return reservation details.
But this was different from leetcode wala any question because here the time slots were fixed
Assume you can only book rooms for a particular day, and time slots can be 0-24
Toh ismein kaise optimize karenge was the question 
Ismein baadme question aaya ki room selection ke liye easy plug and play custome logic likhna hoga toh kaise likhenge
Some other follow up questions, was simple enough

Uber LLD 
Implement a cache that can support different kinds of cache eviction strategies, use generics (so we can give any key and value types), and use whatever design patterns you deem fit for the use case.
I hadn’t ever used generics before XD toh I googled and figured it out on the fly; but more or less ho gaya tha
For different cache eviction strategies I used a strategy pattern; pretty much it
Last me we discussed how Ill make this thread safe
Also, he wanted ke cache ke liye hum koi bhi underlying datastore use kar sakein; toh ek datastore interface bana ke kara sab
Went okay okay 8/10



Coinbase ka subah tha
Usne bhi kuch ajib transaction manager types banwaya
Ek koi store banana hai jismein hum different users ke transactions daal sake saare
Each transaction has user, currency, amount, time
Now we want to expose a function to this store jaha ki koi bhi client kisi bhi field ke uuper query kar sake
Now we want to expose a function to this store jaha ki koi bhi client kisi bhi field ke uuper query kar sake
Ismein I made a function that takes a map of query type: valur as input
Toh you can give
User: Taaru
Currency: USD
And itll return all transactions Taaru made in USD
Optimisation par focus nahi tha, bas design par tha

HLD maine Amazon ka hi diya yar, they asked me to
1.⁠ ⁠Design a file sharing service like dropbox. Maine bahut hi amateur sa design banaya, something like what's given in Alex Xu, but I couldn't get too much into detail cause sirf 30 mins the
2.⁠ ⁠Design a product recommendation system for Amazon's homepage. This was very much like News Feed, kaafi typical question hai, also from Alex Xu. Bas ab comments and likes ki jagah we had product relevance ke kuch signals, and feed ki jagah we had recommendations on a page.
Overall, Amazon ke HLDs are easier to crack, because it's only a 30 min discussion which doesn't leave much room for deep dives

Iske Alawa I have Salesforce
Saare rounds ho gaye, here's how it went
1.⁠ ⁠DSA round, ek BFS ka question tha, ek Top Sort ka question that. Quite straightforward, basic si implementation karke chal gaya, no added complexities
2.⁠ ⁠DSA round, ek priority queue ka question tha, to find k longest strings in a list of strings. Only complexity was to write a custom comparator (v easy), they also allowed me to look at documentation. Ismein the interviewer then extended this question, to ask ki agar strings ki jagah koi aur data type karna hua toh kaise karoge (use generics)? The second question was this https://leetcode.com/problems/search-a-2d-matrix/
3.⁠ ⁠Design / LLD Round, they asked my to write basic machine code for a tiered caching system. L1 -> L2 -> L3. There was no focus on concurrency or anything. Basic requirement thi ki koi bhi lookup pehle L1 mein hoga, vaha nahi milega toh L2 mein and then L3 mein. Maine ek Cache manager class bana ke kar diya, interviewer was not fully satisfied. Interview ke baad maine dekha toh pata chala this is perfect place to apply Chain Of Responsibility wala design pattern; toh design patterns padh ke jaana yaha. Phir kuch time he discussed Kubernetes with me (since I'd added few things related to it on my resume), asked me questions to test my knowledge of K8s
4.⁠ ⁠HM, ismein purely resume par discussion hua, deep dives into past project, standard HM questions. Challenging project batao koi, koi conflict ka example do. Yaha main stories prep karke nahi gaya tha toh pata nahi kaisa gaya hai yeh 🫣 dekhi jayegi ab