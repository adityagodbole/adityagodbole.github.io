---
layout: post
title: FUD - Java verbosity
---

One of the pet FUD items of Go programmers when it comes to Java bashing
is the verbosity of the language.
<!--more-->
Here is a real world comparison. This is a function that takes the name of
a text file which has one number on each line and returns an array of
long/uint64 numbers, or raises exception/returns error on error.

```java
private static List<Long> getNumbers(String filename) throws IOException {
    final List<String> strings = Files.readAllLines(Paths.get(filename));
    return strings.stream()
            .map(Long::parseLong)
            .collect(Collectors.toList());
}
```

```go
func getNumbers(name string) ([]uint64, error) {
	rFile, err := os.Open(name)
	if err != nil {
		return nil, err
	}
	scanner := bufio.NewScanner(rFile)
	nums := []uint64{}
	for scanner.Scan() {
		s := scanner.Text()
		val, err := strconv.ParseUint(s, 10, 64)
		if err != nil {
			return nil, err
		}
		nums = append(nums, val)
	}
	if err := scanner.Err(); err != nil {
		return nil, err
	}
	return nums, nil
}
```

So, which one is verbose and more error prone?
