---
layout: post
type: drafts
title: What I learned with the Dev Summit 2015 talks
category:
---

Testing:

Testing support Library 

flavors prod/mock
- class Injection, returns implementation mocked in mockFlavor and real one in prod
- Different types of tests:
  	- Unit tests (local tests)
    	- test your business logic
    	- mockable Android Jar
    	- Use classes with Mockito
    	- test-driven development
  - Android UI tests
	- Runs on android device or emulator
	- Cloud Test lab
	- Espresso
		- ActivityTestRule (starts and stops activity before each test)
  - Performance Tests (next levels)		
	- EnablePostTestDumpsys
	- EnableNetStatsDump
	- EnambleLogCatDump
	
	- results of battery 
	- janky frames (too long to render)
  - Smarter Testing
	- Scaling in CI tests
- Examples Codelab
  	- Android Testing codelab
  	- Android Perf Testing 
