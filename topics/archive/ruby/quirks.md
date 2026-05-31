---
topic: ruby
status: done
---

# [Pickaxe Book - Programming Ruby 3.3](https://learning.oreilly.com/library/view/programming-ruby-3-3/9798888650684/)

- **alises**: `alias new_name old_name` (can be used to redefine methods while keeping access to the old one)
- **`` ` `` or `%x{}`**: Execute shell commands and return the output. e.g., ``puts `date` `` #=>  Wed Aug 28 22:00:35 PDT 2024
```ruby
​ 	​def `​(cmd)
​ 	  result = old_backquote(cmd)
​ 	  ​unless​ Process.​last_status​.​success?​
​ 	    puts ​"*** Command ​​#{​cmd​}​​ failed: status = ​​#{​Process.​last_status​.​exitstatus​​}​​"​
​ 	  ​end​
​ 	  result
​ 	​end​

​ 	print ​`ls -l /etc/passwd`​
​ 	print ​`ls -l /etc/wibble`​
```

(Imagine writing a parser for Ruby, MUST BE FUN!!!)

**JUST** `nil` and `false` are falsy values in Ruby. not `0` or `""` or `[]` or `{}`.
- **&&** and **and** (returns the first falsy value or the last truthy value):`and` has lower precedence than `&&` and even than `=`. e.g., `x = false and true` will assign `false` to `x`. (crying in Ruby)
- Same goes for **||** and **or**, **not** and **!**.
- **||=**: Assigns a value to a variable if it is not already assigned.
- **!!**: Converts a value to a boolean.
- **`&.`**: Safe navigation operator. e.g., `obj&.method` will return `nil` if `obj` is `nil`.
- **`$.`**: The last line number read from a file. Full list in the [Fandom](https://ruby.fandom.com/wiki/Special_variables) FFS there is a fandom!!!

- Do we really need unless? How about `if not`?
- Also `until` is a thing.

### Pattern Matching

- **pin operator**: `^` to match the value of a variable.
- **`in`**: Used to match the value of a variable.
- **`*`**: Matches any value.(ANYWHERE!!) a,*,b = [1,2,3,4,5] will assign a=1, b=5.
- **`_`**: Matches any value but does not bind it to a variable.
- **`=>`**: Used to bind a value to a variable. e.g., `a => 1` will bind `1` to `a`. I am just glad they did not created top to bottom assignment operator.
```
a
👇
2
```
- **deconstruct_keys**: Used to destructure a hash. e.g., `{a: 1, b: 2} in {a: a, b: b}` will bind `1` to `a` and `2` to `b`.
```ruby
	​def​ ​pick_a_card​(rank_to_look_for, suit_to_look_for, cards)
​ 	  ​case​ cards
​ 	  ​in​ [*, {​rank: ​^rank_to_look_for, suit:}, *]
​ 	    ​"You have a ​​#{​rank_to_look_for​}​​! Its suit is ​​#{​suit​}​​."​
​ 	  ​in​ [*, {rank:, ​suit: ​^suit_to_look_for}, *]
​ 	    ​"You have a {rank}! Its suit is ​​#{​suit_to_look_for​}​​."​
​ 	  ​else​
​ 	    ​"You have no interesting cards,"​
​ 	  ​end​
​ 	​end​
​ 	
​ 	puts pick_a_card(​"King"​, ​"Clubs"​, [
​ 	  {​rank: ​​"Ace"​, ​suit: ​​"Hearts"​},
​ 	  {​rank: ​​"King"​, ​suit: ​​"Diamonds"​},
​ 	  {​rank: ​​"Queen"​, ​suit: ​​"Clubs"​}
​ 	])  You have a pair of Aces.


​ 	​def​ ​pick_a_card​(cards)
​ 	  cards = cards.​sort_by​ { _1[​:rank​] }
​ 	  ​case​ cards
​ 	  ​in​ [{rank:}, {​rank: ​^(rank + 1)}, {​rank: ​^(rank+ 2)}] ​if​ rank > 6
​ 	    ​"You have three consecutive high cards"​
​ 	  ​else​
​ 	    ​"You have no interesting cards,"​
​ 	  ​end​
​ 	​end​
```

- **IO.popen**: when it's called with a "-" it will fork the ruby interpreter and return the object to the `parent` process and `nil` to the `child` process.
The child reads from the parent and writes to the parent using `gets` and `puts` respectively. The parent does the opposite using the `pipe` object.

```ruby
​ 	new_pipe = IO.​popen​(​"-"​,​"w+"​)
​ 	​if​ new_pipe
​ 	  new_pipe.​puts​ ​"Get a job!"​
​ 	  $stderr.​puts​ ​"I'm the parent, the child said to me '​​#{​new_pipe.​gets​.​chomp​​}​​'"​
​ 	​else​
​ 	  $stderr.​puts​ ​"I'm the child, the parent said to me '​​#{​gets.​chomp​​}​​'"​
​ 	  puts ​"OK"​
​ 	​end​

	# Output:
	# I'm the child, the parent said to me 'Get a job!'
	# I'm the parent, the child said to me 'OK'
```
**Tags**: `Ruby 3.3` 