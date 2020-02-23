CLASS LOADERS:
	- BootStrap ClassLoader / Primodial ClassLoader : loads classes from the location rt.jar (/jdk/lib/*.jar   /jre/lib/*.jar).
	- Extension ClassLoader : loads the extensions of core java classes from the respective JDK Extension library (/jre/lib/ext/*.jar OR directory pointed SysVar java.ext.dirs).
	- Application ClassLoader / System ClassLoader :  loads the Application type classes (EnvVar CLASSPATH OR -classpath or -cp command line option)

The ClassLoader follows Delegation Hierarchy Model (Request is delegated to top and then computation and results are flowed to bottom) always functions in the order Application ClassLoader->Extension ClassLoader->Bootstrap ClassLoader. The Bootstrap ClassLoader is always given the higher priority, next is Extension ClassLoader and then Application ClassLoader.

Lets assume there are 3 packages:
	com.arcesium.*
		com.arcesium.one
		com.arcesium.two
	com.yatra.*
		com.yatra.one
		com.yatra.two
	com.company.app -- (imports all the above packages)

Example:
	package com.arcesium.one;

	public class A{
		public void testA(){
			System.out.println("This is A Testing");
		}
	}
	
==> The directory stucture will be
C:
│   A.java
│   B.java
│   AppStart.java
│   C.java
│   D.java
│   arcesium.jar
│   yatra.jar
│
└───com
    ├───arcesium
    │   ├───one
    │   │       A.class
    │   │
    │   └───two
    │           B.class
    │
    ├───company
    │   └───app
    │           AppStart.class
    │
    └───yatra
        ├───one
        │       C.class
        │
        └───two
                D.class

==> To compile the above java code 
javac -d . *.java

Explanition: 
	"-d" is used to specify directory
	"." is to specify current directory
	"*.java" every java file

	The above command checks exsistance of packages in java files and creates respective folder stucture.
	creates the .class files in respective folders.
	
==> In Order to create a seperate jar for each package use below command:
jar cvf arcesium.jar /com/arcesium/one/*.class /com/arcesium/two/*.class
jar cvf yatra.jar /com/yatra/one/*.class /com/yatra/two/*.class

Explanition:
	"jar" this is an exe provided by JDK, similar to javac and java.
	"c" CREATE
	"v" Verbose Output
	"f" Output file name (i.e. jar name)
	"m" Manifest.txt file [If we need to specify main class for creating executable jar]
		Manifest file contains info about the main method that needs to be executed, if its a runnable jar. It also contains timespam of modification.
	"e" If there is only one main class then we can directly specify that class using `e`. [No need to create Manifest file explicitly]
	
==> Compiling using jar
javac -d . -cp "yatra.jar;arcesium.jar" AppStart.java

Explanition:
	"-cp" for specifying the class path.
	";" for seperating the paths in classpath value.

==> Running the AppStart.class file
java -cp "./;yatra.jar;arcesium.jar" com.company.app.AppStart

Explanition:
	"./;yatra.jar;arcesium.jar" this class path has very important role.
	When we specify this class path then application class loader will look for classes in this classapath.
	application class loader will check for provided patha and all the subdirs of that path.
	
	"." it checks in all subdirs of project root for AppStart.
	"yatra.jar;arcesium.jar" the classes inside this jars will be loaded by application class loader.


Samples:
	-> java -cp "yatra.jar;arcesium.jar" com.company.app.AppStart
	REASON: As current path is not specified it will not be able to find AppStart.class.
	
	-> cd ./com
	   java -cp ".;yatra.jar;arcesium.jar" com.company.app.AppStart 		--DONT WORK--		Not able to find yatra.jar and arcesium.jar
	   java -cp ".;../yatra.jar;../arcesium.jar" com.company.app.AppStart	--  WORKS  --		As AppStart will be found even at this level.	
	   java -cp ".\yatra\;..\yatra.jar;..\arcesium.jar" com.company.app.AppStart --DONT WORK--	Not able to find AppStart.class.
	   
	   
==> Few ways to add multiple JARs in Classpath:
- Include the JAR name in CLASSPATH environment variable
- Include name of JAR file in -classpath command line option
- Include the jar name in the Class-Path option in the manifest
- Adding JAR in ext directory e.g. C:\Program Files\Java\jdk1.6.0\jre\lib\ext


==> If two jars has same package name and same class, then in AppTest.class which class is picked from which jar.

JAR1 : testX.jar

	package com.test;
	public class X{
		public void testX(){
			System.out.println("This is X Testing");
		}
	}


JAR2 : testXD.jar	
	
	package com.test;
	public class X{
		public void testX(){
			System.out.println("This is dummy X Testing");
		}
	}
	
AppStart.java

	package com.app;
	import com.test.X;	#Now from where this package is imported ?
	public class AppTest{
		public static void main(String args[]){
			X x = new X();
			x.testX();
		}
	}
	
ANSWER:
	It is dependent on the order of jars we specify in classPath, the first jar that is specified is picked.
	
	-> java -cp ".;testDX.jar;testX.jar" com.app.AppTest
	OUTPUT: This is dummy X Testing
	
	-> java -cp ".;testX.jar;testDX.jar;" com.app.AppTest
	OUTPUT: This is X Testing