```go

package main

import (
	"fmt"
	"math/rand"
)

const p = 0.5
const DefaultMaxLevel = 5

type SLNode struct {
	key, val int
	next     []*SLNode
}

func (s *SLNode) nextNode() *SLNode {
	if len(s.next) == 0 {
		return nil
	}
	return s.next[0]
}

func (s *SLNode) hasNext() bool {
	return s.nextNode() != nil
}

type SkipList struct {
	MaxLevel int
	header   *SLNode
}

func CreateSkipList() *SkipList {
	return &SkipList{MaxLevel: DefaultMaxLevel, header: &SLNode{next: []*SLNode{nil}}}
}

func (s SkipList) randomLevel() (n int) {
	for n = 0; n < s.MaxLevel-1 && rand.Float64() < p; n++ {
	}
	return
}

func (s *SkipList) level() int {
	return len(s.header.next) - 1

}

func (s *SkipList) Insert(key, val int) {
	update := make([]*SLNode, s.level()+1, s.MaxLevel)
	candidate := s.getPath(s.header, update, key)

	if candidate != nil && candidate.key == key {
		candidate.val = val
		return
	}

	newLevel := s.randomLevel()

	if currentLevel := s.level(); newLevel > currentLevel {
		for i := currentLevel + 1; i <= newLevel; i++ {
			update = append(update, s.header)
			s.header.next = append(s.header.next, nil)
		}
	}

	newNode := &SLNode{key, val, make([]*SLNode, newLevel+1, s.MaxLevel)}
	for i := 0; i <= newLevel; i++ {
		newNode.next[i] = update[i].next[i]
		update[i].next[i] = newNode
	}
}

func (s *SkipList) getPath(current *SLNode, update []*SLNode, key int) *SLNode {
	depth := len(current.next) - 1
	for i := depth; i >= 0; i-- {
		for current.next[i] != nil && current.next[i].key < key {
			current = current.next[i]
		}
		if update != nil {
			update[i] = current
		}
	}
	return current.nextNode()
}

func (s *SkipList) Display() {
	for i, node := range s.header.next {
		for node != nil {
			fmt.Printf("%v->", node.val)
			node = node.next[i]
		}
		fmt.Println()
	}
}

func (s *SkipList) Search(key int) (value int, ok bool) {
	candidate := s.getPath(s.header, nil, key)

	if candidate == nil || candidate.key != key {
		return 0, false
	}

	return candidate.val, true
}

func (s *SkipList) Delete(key int) bool {
	update := make([]*SLNode, s.level()+1, s.MaxLevel)
	candidate := s.getPath(s.header, update, key)

	if candidate != nil && candidate.key == key {
		for i := 0; i <= s.level() && update[i].next[i] == candidate; i++ {
			update[i].next[i] = candidate.next[i]
		}
	}

	for s.level() > 0 && s.header.next[s.level()] == nil {
		s.header.next = s.header.next[:s.level()]
	}
	return false
}

func main() {
	s := CreateSkipList()
	s.Insert(3, 3)
	s.Insert(8, 8)
	s.Insert(5, 5)
	s.Insert(7, 7)
	s.Insert(11, 11)
	s.Display()
	fmt.Println("====")
}

```
