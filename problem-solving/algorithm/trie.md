# Trie

* [Trie](#trie)
  * [등장 배경](#등장-배경)
  * [특징](#특징)
  * [예시 코드](#예시-코드)

## 등장 배경

* 정수, 실수형 변수
  * 크기 고정 -> 상수 시간 소요
  * _O(lgN)_
* 문자열 변수
  * 최악 경우 -> 문자열 길이 비례
  * _O(MlgN)_

## 특징

* 노드 구성
  * 포인터 목록(고정 배열)
    * Lower case 혹은 Upper case 구성 -> 26개의 포인터 배열
  * 종료 노드 여부 Boolean 값

* `find()`
  * 찾은 문자열 대응 노드의 종료 노드 여부 확인 X
  * 존재 여부 확인
    * 반환 노드의 `terminal` 참 여부 확인

## 예시 코드

Reference: [leetcode discussion](https://leetcode.com/problems/implement-trie-prefix-tree/discuss/58842/Maybe-the-code-is-not-too-much-by-using-"next26"-C++/294306)

```cpp
struct TrieNode {  
    bool end;  
    TrieNode *children[26];  
  
    TrieNode() {  
        end = false;  
        memset(children, NULL, sizeof(children));  
    }  
};  
  
class Trie {  
public:  
    Trie() {  
        root = new TrieNode();  
    }  
  
    ~Trie() {  
        clear(root);  
    }  
  
    void clear(TrieNode *root) {  
        for (int i = 0; i < 26; i++) {  
            if (root->children[i]) {  
                clear(root->children[i]);  
            }  
        }  
        delete root;  
    }  
  
    /** Inserts a word into the trie. */  
    void insert(string word) {  
        TrieNode *dummy = root;  
        for (char &c: word) {  
            if (!dummy->children[c - 'a']) {  
                dummy->children[c - 'a'] = new TrieNode();  
            }  
            dummy = dummy->children[c - 'a'];  
        }  
        dummy->end = true;  
    }  
  
    /** Returns if the word is in the trie. */  
    bool search(string word) {  
        TrieNode *dummy = root;  
        for (char &c: word) {  
            if (!dummy->children[c - 'a']) {  
                return false;  
            }  
            dummy = dummy->children[c - 'a'];  
        }  
        return dummy && dummy->end;  
    }  
  
    /** Returns if there is any word in the trie that starts with the given prefix. */  
    bool startsWith(string prefix) {  
        TrieNode *dummy = root;  
        for (char &c: prefix) {  
            if (!dummy->children[c - 'a']) {  
                return false;  
            }  
            dummy = dummy->children[c - 'a'];  
        }  
        return dummy;  
    }  
  
private:  
    TrieNode *root;  
};
```
