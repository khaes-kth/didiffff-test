# Testing Didiffff on Time-11
This repo includes Time-11 bugs from Defects4J and a patch for that bug. 
The patch is a plausible patch generated by the Arja repair tool. In this
readme, we explain how to run didiffff for this patch and get the results.

## Step 1
Create the experiment directory and cd into it:
```
mkdir didiffff-experiment
cd didiffff-experiment
```

## Step 2
Clone this repo and didiffff repo and configure didiffff 
(you should have the 
[requirements](https://github.com/tetsuyakanda/didiffff/tree/letsgo/sample#requirements) 
installed):
```
git clone git@github.com:tetsuyakanda/didiffff.git
git clone git@github.com:khaes-kth/didiffff-test.git

cd didiffff/kurakuraberuberu
npm install
cd ../didiffff
npm install

cd ../sample
curl -OL https://github.com/takashi-ishio/selogger/releases/download/v0.2.3/selogger-0.2.3.jar
cp ../nod4j-0.2.3-t.jar .
```

## Step 3
Copy the original and patched versions of Time-11 into the sample directory of 
didiffff:
```
# we are at didiffff/sample
cp -r ../../didiffff-test/Time-11-original ./
cp -r ../../didiffff-test/Time-11-patched ./ 
```

These two versions of the project have set selogger as the javaagent [here](https://github.com/khaes-kth/didiffff-test/blob/master/Time-11-original/pom.xml#L245) 
and [here](https://github.com/khaes-kth/didiffff-test/blob/master/Time-11-patched/pom.xml#L245).

Also, the `org.joda.time.tz.TestCompiler` class is modified to only include a test 
that fails on the original version and passes on the patched version (see [here](https://github.com/khaes-kth/didiffff-test/blob/master/Time-11-original/src/test/java/org/joda/time/tz/TestCompiler.java)).

The patch looks like this:
```
@@ -369,7 +369,7 @@
                 millis = next.getMillis();
                 saveMillis = next.getSaveMillis();
                 if (tailZone == null && i == ruleSetCount - 1) {
-                    tailZone = rs.buildTailZone(id);
+                    System.out.println("Writing ZoneInfoMap");
                     // If tailZone is not null, don't break out of main loop until
                     // at least one more transition is calculated. This ensures a
                     // correct 'seam' to the DSTZone.

```

## Step 4
Get the traces for both versions. First, for the original version and then for 
the patched version:

```
cd Time-11-original
mvn test -Dtest="org.joda.time.tz.TestCompiler"
cd ../
java -jar nod4j-0.2.3-t.jar Time-11-original/ selogger/ ../didiffff/public/assets/proj1

rm selogger/ -rf

cd Time-11-patched
mvn test -Dtest="org.joda.time.tz.TestCompiler"
cd ../
java -jar nod4j-0.2.3-t.jar Time-11-patched/ selogger/ ../didiffff/public/assets/proj2
```

## Step 5
Run npm commands on didiffff:
```
cd ../didiffff
npm run load
npm run start
```

## Step 6
Check out the results. Go to `http://localhost:3000`
 and open `Time-11/src/main/java/org/joda/time/tz/DateTimeZoneBuilder.java`.
