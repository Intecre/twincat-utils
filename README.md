# README #

This is a library of utility functions and classes for Beckhoff's TwinCAT.  They may well be transferrable to Codesys platform, where linked libraries permit.

### What is this repository for? ###

* Additional String functions (concatenation)
* Simple logging.  
    * Logs are written to the windows system log for persistence.
* Wrapping _Taskinfo for intuitive access.
* TryParse functions for LREAL, UDINT and WORD. 
    * Mimics C# TryParse, returns TRUE if successful, placeholder passed as VAR_IN_OUT
* String and UDINT lists. 
    * Plan is to refactor a generic underlying list class for these
* Assert class for basic PLC testing.

### How do I get set up? ###

* Download the code.
* Either compile into a library and distribute.
* OR
* Include in the base code for the project.
* Developed on TwinCAT 3.1.4020.28.

### Contribution guidelines ###

* All comments always welcome.
* If you submit a pull/merge request, please include a link to a your test project repo. 
* Include any special instructions I need to re-run your test.

### Who do I talk to? ###

* Me (hello!) - Mark Lazarides