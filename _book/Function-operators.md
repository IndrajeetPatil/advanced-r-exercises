# Function operators



### Exercises 11.2.3

**Q1.** Base R provides a function operator in the form of `Vectorize()`. What does it do? When might you use it?

**Q2.** Read the source code for `possibly()`. How does it work?

**Q3.** Read the source code for `safely()`. How does it work?

### Exercises 11.3.1

**Q1.** Weigh the pros and cons of `download.file %>% dot_every(10) %>% delay_by(0.1)` versus `download.file %>% delay_by(0.1) %>% dot_every(10)`.

**Q2.** Should you memoise `file.download()`? Why or why not?

**Q3.** Create a function operator that reports whenever a file is created or deleted in the working directory, using `dir()` and `setdiff()`. What other global function effects might you want to track?

**Q4.** Write a function operator that logs a timestamp and message to a file every time a function is run.

**Q5.** Modify `delay_by()` so that instead of delaying by a fixed amount of time, it ensures that a certain amount of time has elapsed since the function was last called. That is, if you called `g <- delay_by(1, f); g(); Sys.sleep(2); g()` there shouldn't be an extra delay.