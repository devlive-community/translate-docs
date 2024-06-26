[TOC]

SpotBugs allows several aspects of the analyses it performs to be customized. System properties are used to configure these options. This chapter describes the configurable analysis options.

The analysis options have two main purposes. First, they allow you to inform SpotBugs about the meaning of methods in your application, so that it can produce more accurate results, or produce fewer false warnings. Second, they allow you to configure the precision of the analysis performed. Reducing analysis precision can save memory and analysis time, at the expense of missing some real bugs, or producing more false warnings.

The analysis options are set using the `-property` command line option. For example:

```bash
$ spotbugs -textui -property "cfg.noprune=true" myApp.jar
```

The list of configurable analysis properties is shown in following table:

| Property Name | Value | Meaning|
| --- | --- | --- |
| findbugs.assertionmethods| Comma-separated list of fully qualified method names: e.g., “com.foo.MyClass.checkAssertion” | This property specifies the names of methods that are used to check program assertions. Specifying these methods allows the null pointer dereference bug detector to avoid reporting false warnings for values which are checked by assertion methods. |
| findbugs.de.comment| true or false| If true, the DroppedException detector scans source code for empty catch blocks for a comment, and if one is found, does not report a warning. |
| findbugs.maskedfields.locals| true or false | If true, emit low priority warnings for local variables which obscure fields. Default is false. |
| findbugs.nullderef.assumensp | true or false | not used (intention: If true, the null dereference detector assumes that any reference value returned from a method or passed to a method in a parameter might be null. Default is false. Note that enabling this property will very likely cause a large number of false warnings to be produced.)|
| findbugs.refcomp.reportAll | true or false | If true, all suspicious reference comparisons using the == and != operators are reported.,If false, only one such warning is issued per method.,Default is false. |
| findbugs.sf.comment | true or false | If true, the SwitchFallthrough detector will only report warnings for cases where the source code does not have a comment containing the words “fall” or “nobreak”. (An accurate source path must be used for this feature to work correctly.) This helps find cases where the switch fallthrough is likely to be unintentional. |
