They recommend you read this https://nasa.github.io/fpp/fpp-users-guide.html#Defining-Types then this https://nasa.github.io/fpp/fpp-spec.html
  - GDS limitations 
	  - F Prime still uses the old XML format that they used to in order to support the GDS. 
		  - Some FPP features aren't supported by the old XML format, so limitations will be noted when necessary 
		  - Because of this, some FPP features aren't supported at all by the GDS yet, this will also be pointed out when needed
  - FPP - F prime prime - Modeling Language
	  - `fpp-check file.fpp` - tool to check if an FPP model is valid - can also use standard in: `fpp-check < file.fpp` - no output means the provided code is good
		  - without any given parameters it'll prompt for you to type in code, then you press ctrl+D to have it check the code 
	  - constants
		  - to use a reserved word (such as `constant`) as a constant's name, you must put the '$' character in front of the name
			  - ex. `$constant` is a valid name
		  - Strings
			  - escape sequence for Strings = `\`
				  - can do `\"` to display quotation marks
			  - multiline strings use triple quotes - `"""`
		  - Arrays - can have at most 256 elements
			  - cannot put different types into the same array, but if they're easily convertible (such as an int and a float), then it's fine
			  - if you're not sure if type conversion is allowed, just check using fpp-check
		  - structs - struct value expression; represents a C or C++ style structure, such as a mapping of names to values
			  - struct member consists of a name, equal sign, and value - `x = 1`
			  - order of members doesn't matter
			  - can use structs in arrays, just using curly braces to denote it's a struct
				  - `constant something = [{x=2, y=1}, {z=1, u=12}]`
		  - order of constant declarations don't matter, thus the following code is fine:
			```
			const a = b;
			const b = 2;
			```
		 - you can separate code by semicolons, or simply by newlines 
		 - can create an explicit line continuation by using \
	 - Comments & annotations
		 - comments begin with a `#` and are only displayed in the FPP file
		 - annotations being with either an `@` or `@<` and are displayed in the generated C++ file - everything after the `@` or `@<` is considered an annotation until the newline
			 - `@` denotes a pre-annotation - annotates the element that follows it
			 - `@<` denotes a post-annotation - annotates the element that's before it
	 - Modules - group of elements - basically used for organizational purposes; used in place of object oriented programming kinda
		 - to access constants declared in a module, you do `ModuleName.constantName`
		 - example:
		```
		module M {
			constant a = 1;
		}
		constant b = M.a;
		# constant b will be equal to 1
		```
		 - you can define modules within other modules, allowing for something like `A.B.c` to be legal
			 - this is accessing constant `c` which is within module `B` which is within module `A`
		 - XML LIMITATION - cannot do nested modules, thus nested modules aren't translated properly to C++ as a workaround, just name the module differently
			 - EX. instead of `module A { module B....` instead write `module A_B {....`
	 - Type definitions - may appear at the top level or inside a module definition - annotatable definition
		 - three kinds of type definitions - Array, Struct, and Abstract
		 - array type - describes the element type and its size 
			 - example: `array A = [3] U32` - array named `A` with 3 values, all of which are unsigned 32-bit integers 
			 - the `size` can be a previously declared constant, just has to be a value greater than 0 and less than or equal to 256
			 - string arrays need to know the maximum size of a string, uses a default maximum size if not specified 
				 - ex. `array A = [3] string size 40` - the maximum amount of characters a string can have is 40
			 - array definitions can have another array definition as its element type, but an array cannot use itself as its type
		 - 