---
layout:     post
title:      一个好玩的排序
date:       2015-01-12 23:33:00
summary:    sleep sort
published:  true
categories: sort golang

---

今天翻[Study Golang](http://studygolang.com)，看到一个好玩的排序算法，具体就不说了，看码自然懂。

{% highlight go %}
package main

import (
	"fmt"
	"math/rand"
	"sort"
	"sync"
	"time"
)

type Case struct {
	origin []int
	sorted []int
}

func (self *Case) Len() int {
	return len(self.origin)
}

func (self *Case) Less(i, j int) bool {
	return self.sorted[i] <= self.sorted[j]
}

func (self *Case) Swap(i, j int) {
	self.sorted[i], self.sorted[j] = self.sorted[j], self.sorted[i]
}

func (self *Case) Display() {
	fmt.Printf("origin:")
	for _, i := range self.origin {
		fmt.Printf(" %d", i)
	}
	fmt.Printf("\nsorted:")
	for _, i := range self.sorted {
		fmt.Printf(" %d", i)
	}
	fmt.Println()
}

func InitRandomCase(length int) Case {
	cas := Case{}
	cas.origin = make([]int, length)
	cas.sorted = make([]int, length)
	for i := 0; i < length; i++ {
		rand.Seed(time.Now().UnixNano())
		randn := rand.Intn(100)
		cas.origin[i] = randn
		cas.sorted[i] = randn
	}
	sort.Sort(&cas)
	cas.Display()
	return cas
}

func (self *Case) CheckSort(sortFunc SortFunc) bool {
	res := sortFunc(self.origin)
	if len(res) != len(self.sorted) {
		return false
	}
	for i, v := range res {
		if v != self.sorted[i] {
			return false
		}
	}
	return true
}

func mkCases(length, count int) []Case {
	cases := make([]Case, count)
	for i := 0; i < count; i++ {
		cas := InitRandomCase(length)
		cases = append(cases, cas)
	}
	return cases
}

func main() {
	cases := mkCases(10, 4)
	for i, cas := range cases {
		fmt.Printf("%d case pass? %v\n", i, cas.CheckSort(SleepSort))
	}
}

type SortFunc func([]int) []int

func SleepSort(lst []int) []int {
	var res []int
	var wg sync.WaitGroup
	for _, item := range lst {
		wg.Add(1)
		go func(i int) {
			time.Sleep(time.Duration(i) * time.Millisecond)
			res = append(res, i)
			wg.Done()
		}(item)
	}
	wg.Wait()
	return res
}
{% endhighlight %}
