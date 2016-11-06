<strong><strong>1. Pure Functions</strong></strong>

This one is really simple. You already know and understand it. A pure function accepts an input and returns an output. <em><em>It does nothing else.</em>  </em>It does not mutate data.  It has no side effects.<em>  </em>This is all that is meant by its purity.

<em>Pure Function: Addition</em>


````ruby
first_num = 5
second_num = 12

first_num + second_num
=> 17

first_num
=> 5

second_num
=> 12
````


See -- nothing changed in the environment.

<em>Impure Function (mutated data):</em> `map!`


````ruby
some_cool_numbers = [1, 3, 5, 7, 11, 13]
some_cool_numbers.map! { |number| number**2 }

some_cool_numbers
=> [1, 9, 25, 49, 121, 169]
````

Whenever you call `map!` something in the environment will be different
after the call than it was before the call.

This is great! This is not terrible! Do not give up `map!`, or any other
'bang' functions. They are useful.

<em>Impure Function (side effect):</em> `each`

````ruby
sum = 0

some_cool_numbers = [1, 3, 5, 7, 11, 13]

some_cool_numbers.each { |number| sum += number }

sum
=> 40

some_cool_numbers
=> [1, 3, 5, 7, 11, 13]
````

This is a side effect. Something, somewhere else, changed. The input to the function is not the thing that changed. Side effects are also useful. Don't give them up. But, with great convenience comes great debugging headaches. Figuring out what happened when something goes wrong here is more annoying. You can't look just at the input and the output. You also have to look at the environment. Mo' things to keep in your head, mo' headache.

That's it!  All anyone means by a pure function is a function that neither directly nor indirectly (through side effects) mutates data.

<strong>2. An Idiot Hangs Himself</strong>

Before enrolling in App Academy (blah, blah, blah -- we can talk about that elsewhere), I had to do some prep work.  One assignment was a simple program that would let a human play hang-man against a computer. In order to make the computer a better opponent I fed it a dictionary and had it get the letter frequencies of reasonable guesses (guesses of correct length, that contained all the right guesses, none of the wrong guesses, etc.). Knowing that I would want to refer to these frequencies several times, and not wanting to re-calculate them each time, I created an instance variable to store this frequency table. Then, I wrote a <code>get_frequencies</code> function that would set the value
of this variable whenever I ran it.

````ruby
def get_frequencies(words)
  frequencies = Hash.new(0)
    words.each do |word|
      word.chars.each do |letter|
        unless @guessed_letters.include?(letter)
        frequencies[letter] += 1
      end
    end
  end
  @letter_frequencies = frequencies
end
````

'Ha -- I am so smart', I thought. 'I am saving computation power'. And I was. I could refer to that table whenever I wanted without re-computing it. But, I was wrong. I chose the wrong trade off.

This is fine. I was just starting out. I made a decision with some rationale. If asked, I could state the reason. I don't think I was being an idiot for doing this, even though I was wrong (see below). I don't think you are being an idiot for choosing the wrong trade off either.

<strong>3. Obvious Headaches</strong>

This was a very bad design decision. I was now unable to calculate any letter frequencies without updating an unrelated instance variable. 'Why would I ever need to calculate letter frequencies outside of this one situation I am thinking about right now?' I thought with my original narrow focus.[1] 'Because,' anyone without that narrow focus would be able to respond, 'you'll want to get a frequency table of the board as it currently exists and compare it to any potential guess. In order to do that, you will have to write a new function to get the frequencies of a single word.'

And they were right. That is what I did.

````ruby
def get_frequencies_single_word(word)
  frequencies = Hash.new(0)
  word.chars.each do |letter|
    frequencies[letter] += 1
  end
  frequencies
end
````

This code is almost identical to `get_frequencies(words)` except that 1) it operates on a string and 2) it has no side effects (it is a pure function). This is annoying. It makes my class longer and I don't want to look at a long class. The method is functionally similar enough that it has a similar name, meaning my auto-complete functions are more irritating to use. I am liable to make stupid errors by using the wrong method. All of this is stuff that I'll have to either deal with for the life of the program, or refactor to get rid of.[2]  I assume you'd rather not deal with it either.

<strong>4. Pure Function Solution</strong>

This was an easy and unforced error. There is an obvious solution to this.

````ruby
def get_frequencies(words)
  frequencies = Hash.new(0)
  words.each do |word|
    word.chars.each do |letter|
      unless @guessed_letters.include?(letter)
        frequencies[letter] += 1
      end
    end
  end
  # Just change this next line. That's it!
  # @letter_frequencies = frequencies
  # instead just return frequencies like this:
  frequencies
end
````

The new function is :

````ruby
def get_frequencies(words)
  frequencies = Hash.new(0)
  words.each do |word|
    word.chars.each do |letter|
      unless @guessed_letters.include?(letter)
        frequencies[letter] += 1
      end
    end
  end
  frequencies
end
````

This is a pure function. It takes an input. It creates the output within itself. It returns that output. Nothing else is changed by the function. There are no side effects, and the input is not modified. It does everything my other two functions did[3]. All I have to do is change the code so that it's called within the appropriate variable assignment, like so:
 `get_frequencies(words)` becomes:
`@letter_frequencies = get_frequencies(words)`.

In other words, all you do is take all of the mutation outside of the function itself. Your functions are far more useful now.

<strong>5. Trade-Off: Uglier Code</strong>

Ruby people pride themselves in having code that is easy on the eyeballs.

`get_frequencies(words)`

is easier to look at than:

`@letter_frequencies = get_frequencies(words)`

We are humans. We read prose. Lots of assignments in a row are more irritating to read than lots of (well named) function calls in a row. There is less stuff for your brain to parse. Don't believe me? Look a this:

````ruby
def handle_response(letter, indicies)
  if @board == []
    @secret_length.times { @board << "-"}
  end
  unless indicies == []
    indicies.each do |index|
      @board[index] = letter
    end
  end
  @guessed_letters << letter
  filter_candidate_words
  end
end
````

vs.

````ruby
def handle_response(letter, indicies)
  create_board if is_board_empty?
  update_board_value
  update_guessed_letters
  filter_candidate_words
end
````

The former is a lot more difficult to look a than the latter.

You can have the best of both worlds by creating a lot of additional wrapper functions like `get_frequencies_and_assign_to_frequency_table`. But, that takes time and adds lines to your code. It's another trade off. Is it worth it? This is a judgement call. It's also one that I don't currently have enough experience to make (10/16).

But, you know, for a beginner just banging around, simply recognizing these concerns is a good thing. Please don't shame anyone for making a sub-optimal design decision. Find out why they did, explain why you don't like it, and then try to figure out what the best thing to do is.

 ______

[1] This is why learning good technique is helpful. Whenever we focus on a problem, we narrowly focus on that problem. This is a good thing.  We would be paralyzed otherwise.  We are unable to think of ever possible ramification of any decision we make.  Even most deities can't do that. Good techniques are often good because they allow us to proceed without making design decisions that will restrict our options in the future, and they purchase this design-delay at minimal costs.

[2] Why did I write this new function rather than re-factor? Because I had a time deadline. I could have just fixed the original function, but it was called about seven [I just made this number up to illustrate the point -- don't yell a me] times throughout the program. I would have had to alter or update several of those. This would take time that I didn't have.

[3] This is a lie. I would have to call to_a on the input so that, if it is a single word, I can still call an enumerable on it. Or, I would have to remember to turn all of my inputs into arrays beforehand.
