# How to use github's search functionality to find code: MOM6 as an example

Presenter: @dougiesquire

Searching [the MOM6 code repository in the ACCESS-NRI GitHub org](https://github.com/access-nri/mom6).
Contains all of the MOM6 Fortran code used in ACCESS-OM3

## Quick summary of GitHub search functionality

Pros:

 - Easy and convenient
 - Allows you to see how parts of the model work

Cons:

 - Can only search on the default branch of the repo.
 - GitHub doesn't actually index everything.


## More detailed notes

- Why use the GitHub search bar?
    - See how a specific piece of code is written - how is it implemented in MOM6? Need to search through the code
- Dougie’s screen: Looking at [MOM6 fork on ACCESS-NRI GitHub org](https://github.com/ACCESS-NRI/MOM6)
    - Can click on search at top or click `/` to start a search
    - Eg [search `global`](https://github.com/search?q=repo%3AACCESS-NRI%2FMOM6%20global&type=code)
        - Will return all files in this repo with “Global” (not case sensitive by default)
        - Get the summary of the files, and can click to expand more lines of code
        - Can click on the line itself and it will take you to that line in the file
    - Can search on multiple terms
        - Eg `Global Mean` - searches all files with global or mean
        - Eg `“Global Mean”` - search for specific phrase
        - Eg `“Global mean ocean salinity”` - can search directly for this diagnostic
            - Finds where it’s registered, and can backtrack to find the code
    - Can search on patterns (regex) rather than words
        - Eg search `Global` but only at specific path: `Global path:src/core`
    - Fast, powerful, easy to use
    - Can share searches with people (can’t do this if you are, eg, using “grep” on Gadi terminal)
- Gotchas
    - Can only search on default branch
    - GitHub doesn’t index everything, and opaque which code is indexed
        - Eg doesn’t index vendor code, but not clear what that means
        - If you search something and it seems weird that nothing is found, then this might be the main issue

### Questions from the audience
- _Q: Can it organize the search in order of the subroutines called?_
    - MOM6 docs does include schematics on subroutines that are called from specific file
        - But may not be generated anymore, so may be out of date
- _Q: Can searches be case sensitive?_
    - Yes, you can use regex: `/(?-i)Global/`
    - By default they aren’t case sensitive
- _Q: What is regex?_
    - Language for pattern matching
    - Stands for regular expression
    - Lots of resources online
        - Just be sure to use the correct version of regex, as there are different types
    - Can click “Search syntax tips” - link at bottom of search popup in GitHub - provides quick help for lots of common regex uses
- _Q: OM3 configurations are all on a branch - does that mean we’ve eliminated the ability to search through OM3 configurations?_
    - Yes, we can’t search OM3 configurations at the moment using this tool, because there is almost nothing on the default branch. This is definitely a limitation.
    - Some parameter documentation has been moved to main branch specifically to be able to search on them
    - There are other ways to search through other branches, but no examples were given
    - Lots of people would like GitHub to add functionality to search on a branch other than the default, but not the case currently
 
## More resources
[Understanding GitHub Code Search syntax](https://docs.github.com/en/search-github/github-code-search/understanding-github-code-search-syntax#query-for-an-exact-match)
