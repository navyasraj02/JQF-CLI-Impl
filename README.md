# Continuous Fuzzing for Java Example

This is an example of how to integrate your [JQF](https://github.com/rohanpadhye/jqf) targets into GitLab CI/CD
[Ref Link](https://gitlab.com/gitlab-org/security-products/demos/coverage-fuzzing/java-fuzzing-example/-/tree/master?ref_type=heads)

## Building JQF Target

### Understanding the bug

The bug is located at `ParseComplex.Java` in the following code

```java
package org.gitlab.examplejava;

public class ParseComplex {
    public static boolean parse(String data) {
        if (data.length() > 4) {
            return false;
        }
        return data.charAt(0) == 'F' &&
                data.charAt(1) == 'U' &&
                data.charAt(2) == 'Z' &&
                data.charAt(3) == 'Z';
    }
}
```

This is a VERY simple case for the sake of the example the author made a mistake.
and Instead of `data.length() > 4 ` the correct code should be `data.length() < 4`.

### Understanding the fuzzer

the fuzzer is located at `ParseComplexFuzz.Java` in the following code:

```java
import org.junit.runner.RunWith;
import edu.berkeley.cs.jqf.fuzz.Fuzz;
import edu.berkeley.cs.jqf.fuzz.JQF;

@RunWith(JQF.class)
public class ParseComplexFuzz {

    @Fuzz
    public void fuzz(String data) {
        ParseComplex.parse(data);
    }
}
```

This is pretty straight forward the fuzzer will generate the psudo random string via data according to
the coverage feedback

### Steps to run

```bash
1. git clone https://gitlab.com/gitlab-org/security-products/demos/coverage-fuzzing/java-fuzzing-example.git
2. cd java-fuzzing-example
3. mvn package
4. java -jar zest-cli.jar -e ./target/example-java-1.0-SNAPSHOT-fat-tests.jar dev.fuzzit.examplejava.ParseComplexTest fuzz
```

Will print the following output and stacktrace:

```text
Semantic Fuzzing with Zest
--------------------------

Test name:            org.gitlab.examplejava.ParseComplexTest#fuzz
Results directory:    /app/target/fuzz-results/dev.gitlab.examplejava.ParseComplexTest/fuzz
Elapsed time:         2s (no time limit)
Number of executions: 582
Valid inputs:         562 (96.56%)
Cycles completed:     0
Unique failures:      1
Queue size:           2 (0 favored last cycle)
Current parent input: 0 (favored) {581/800 mutations}
Execution speed:      263/sec now | 221/sec overall
Total coverage:       5 branches (0.01% of map)
```

You can see 1 crash is instantly found. Results are saved by default to target/fuzz-results


```text
.id_000000: FAILURE (java.lang.StringIndexOutOfBoundsException)
E
Time: 0.079
There was 1 failure:
1) fuzz(org.gitlab.examplejava.ParseComplexTest)
java.lang.StringIndexOutOfBoundsException: String index out of range: 0
        at java.base/java.lang.StringLatin1.charAt(StringLatin1.java:47)
        at java.base/java.lang.String.charAt(String.java:702)
        at org.gitlab.examplejava.ParseComplex.parse(ParseComplex.java)
        at org.gitlab.examplejava.ParseComplexTest.fuzz(ParseComplexFuzz.java)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.base/java.lang.reflect.Method.invoke(Method.java:567)
        at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
        at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
        at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
        at edu.berkeley.cs.jqf.fuzz.junit.TrialRunner$1.evaluate(TrialRunner.java:59)
        at edu.berkeley.cs.jqf.fuzz.junit.TrialRunner.run(TrialRunner.java:65)
        at edu.berkeley.cs.jqf.fuzz.junit.quickcheck.FuzzStatement.evaluate(FuzzStatement.java:165)
        at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
        at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
        at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
        at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
        at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
        at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
        at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
        at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
        at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
        at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
        at edu.berkeley.cs.jqf.fuzz.junit.GuidedFuzzing.run(GuidedFuzzing.java:184)
        at edu.berkeley.cs.jqf.fuzz.junit.GuidedFuzzing.run(GuidedFuzzing.java:126)
        at edu.berkeley.cs.jqf.plugin.ReproGoal.execute(ReproGoal.java:184)
        at org.apache.maven.plugin.DefaultBuildPluginManager.executeMojo(DefaultBuildPluginManager.java:137)
        at org.apache.maven.lifecycle.internal.MojoExecutor.execute(MojoExecutor.java:210)
        at org.apache.maven.lifecycle.internal.MojoExecutor.execute(MojoExecutor.java:156)
        at org.apache.maven.lifecycle.internal.MojoExecutor.execute(MojoExecutor.java:148)
        at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject(LifecycleModuleBuilder.java:117)
        at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject(LifecycleModuleBuilder.java:81)
        at org.apache.maven.lifecycle.internal.builder.singlethreaded.SingleThreadedBuilder.build(SingleThreadedBuilder.java:56)
        at org.apache.maven.lifecycle.internal.LifecycleStarter.execute(LifecycleStarter.java:128)
        at org.apache.maven.DefaultMaven.doExecute(DefaultMaven.java:305)
        at org.apache.maven.DefaultMaven.doExecute(DefaultMaven.java:192)
        at org.apache.maven.DefaultMaven.execute(DefaultMaven.java:105)
        at org.apache.maven.cli.MavenCli.execute(MavenCli.java:956)
        at org.apache.maven.cli.MavenCli.doMain(MavenCli.java:288)
        at org.apache.maven.cli.MavenCli.main(MavenCli.java:192)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.base/java.lang.reflect.Method.invoke(Method.java:567)
        at org.codehaus.plexus.classworlds.launcher.Launcher.launchEnhanced(Launcher.java:282)
        at org.codehaus.plexus.classworlds.launcher.Launcher.launch(Launcher.java:225)
        at org.codehaus.plexus.classworlds.launcher.Launcher.mainWithExitCode(Launcher.java:406)
        at org.codehaus.plexus.classworlds.launcher.Launcher.main(Launcher.java:347)

FAILURES!!!
Tests run: 1,  Failures: 1

```
