### Problem

Longest Palindromic Substring

Given a string s, find the longest palindromic substring in s. You may assume that the maximum length of s is 1000.

### Solution

#### Dynamic Programming

![Dynamic Programming](/images/longest-palindromic-substring-dp.png)


```go
package main

var (
	maxLen  = 1
	startAt = 0
	endAt   = 1
)

func main() {
	println(longestPalindrome("abba1"))
}

// Dynamic Programming
func longestPalindrome(s string) string {
	if len(s) < 2 {
		return s
	}

	runes := []rune(s)
	for i := range runes {
		extendPalindrome(runes, i, i)
		extendPalindrome(runes, i, i+1)
	}

	return string(runes[startAt:endAt])
}

func extendPalindrome(r []rune, j int, k int) {
	for j >= 0 && k < len(r) && r[j] == r[k] {
		j--
		k++
	}

	if k-j-1 > maxLen {
		maxLen = k - j - 1
		startAt = j + 1
		endAt = k
	}
}
```
#### TODO OTHER SOLUTION
