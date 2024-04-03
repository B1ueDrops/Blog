---
title: 用Rust实现Leetcode Hot100
categories: 编程语言
---





## 两数之和

```rust
use std::collections::HashMap;

impl Solution {
    pub fn two_sum(nums: Vec<i32>, target: i32) -> Vec<i32> {

        let mut hash: HashMap<i32, i32> = HashMap::new();

        for (i, v) in nums.iter().enumerate() {
            if let Some(index) = hash.get(&(target - *v)) {
                return vec![i as i32, index.to_owned()];
            }
            hash.insert(v.to_owned(), i as i32);
        }
        return vec![];
    }
}
```



## 字母异位词分组

```rust
use std::collections::HashMap;

impl Solution {
    pub fn group_anagrams(strs: Vec<String>) -> Vec<Vec<String>> {
        
        let mut hash: HashMap<String, Vec<String>> = HashMap::new();
        for str in &strs {
            let mut cv: Vec<char> = str.chars().collect();
            cv.sort();
            let os = cv.iter().collect::<String>();
            let vec: Vec<String> = Vec::new();
            let mut val = hash.entry(os).or_insert(vec);
            val.push(str.to_owned());
        }

        let mut ans: Vec<Vec<String>> = Vec::new();
        for vec in hash.values() {
            ans.push(vec.to_owned());
        }
        return ans;
    }
}
```



## 最长连续序列

```rust
use std::cmp::max;
use std::collections::HashSet;

impl Solution {
    pub fn longest_consecutive(nums: Vec<i32>) -> i32 {
        
        let mut hash: HashSet<i32> = HashSet::new();
        for x in nums {
            hash.insert(x.to_owned());
        }
        let mut ans: i32 = 0;
        for x in &nums {
            if !hash.contains(&(*x - 1)) {
                let mut y = x.to_owned();
                while hash.contains(&(y + 1)) {
                    y = y + 1;
                }
                ans = max(ans, y - x + 1);
            }
        }
        return ans;
    }
}
```



## 无重复字符的最长子串

```rust
use std::cmp::max;
use std::collections::HashMap;

impl Solution {
    pub fn length_of_longest_substring(s: String) -> i32 {
        
        let mut hash: HashMap<char, i32> = HashMap::new();
        let mut j: usize = 0;
    
        let mut ans: usize = 0;

        for (i, c) in s.chars().enumerate() {
            if hash.get(&c).is_none() {
                hash.insert(c, 1);
            } else {
                *hash.entry(c).or_insert(0) += 1;
            }
            while j < i {
                let ith_char = s.chars().nth(i).unwrap();
                let ith_char_cnt = hash.get(&ith_char).unwrap();
                if *ith_char_cnt <= 1 {
                    break;
                }

                let jth_char = s.chars().nth(j).unwrap();
                *hash.entry(jth_char).or_insert(0) -= 1;
                j = j + 1;
            }
            ans = max(ans, i - j + 1);
        }
        ans as i32
    }
}
```

