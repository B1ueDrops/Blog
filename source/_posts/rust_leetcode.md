---
title: 用Rust实现Leetcode算法题
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



## 移动零

```rust
impl Solution {
    pub fn move_zeroes(nums: &mut Vec<i32>) {
        let mut k = 0;
        for i in 0..nums.len() {
            if nums[i] != 0 {
                nums[k] = nums[i];
                k = k + 1;
            }
        }
        while k < nums.len() {
            nums[k] = 0;
            k = k + 1;
        }
    }
}
```



## 盛水最多的容器

```rust
use std::cmp::{ min, max };

impl Solution {
    pub fn max_area(height: Vec<i32>) -> i32 {

        let mut res = 0;
        let mut i = 0 as usize;
        let mut j = height.len() - 1;

        while i < j {
            res = max(res, min(height[i], height[j]) * (j - i) as i32);
            if height[i] < height[j] {
                i = i + 1;
            }
            else {
                j = j - 1;
            }
        }
        res
    }
}
```



## 三数之和

```rust
impl Solution {
    pub fn three_sum(nums: Vec<i32>) -> Vec<Vec<i32>> {

        let n = nums.len();
        let mut res: Vec<Vec<i32>> = vec![];
        let mut nums = nums;
        nums.sort();

        for i in 0..nums.len() {
            if i != 0 && nums[i] == nums[i - 1] {
                continue;
            }
            let mut j = i + 1;
            let mut k = n - 1;

            while j < k {
                let t = nums[i] + nums[j] + nums[k];
                if t > 0 {
                    k = k - 1;
                    continue;
                }
                else if t < 0 {
                    j = j + 1;
                    continue;
                }
                else {
                    res.push(vec![nums[i], nums[j], nums[k]]);
                }
                j = j + 1;
                k = k - 1;
                while j < k && nums[j] == nums[j - 1] {
                    j = j + 1;
                }
                while j < k && nums[k] == nums[k + 1] {
                    k = k - 1;
                }
            }
        }
        res
    }
}
```



## 两数相加

```rust
impl Solution {
    pub fn add_two_numbers(l1: Option<Box<ListNode>>, l2: Option<Box<ListNode>>) -> Option<Box<ListNode>> {

        let mut head = Some(Box::new(ListNode::new(-1)));
        let mut cur = &mut head;
        let mut t = 0;
        let mut cur1 = &l1;
        let mut cur2 = &l2;

        while cur1.is_some() || cur2.is_some() || t != 0 {
            if cur1.is_some() {
                t = t + cur1.as_ref().unwrap().val;
                cur1 = &cur1.as_ref().unwrap().next;
            }
            if cur2.is_some() {
                t = t + cur2.as_ref().unwrap().val;
                cur2 = &cur2.as_ref().unwrap().next;
            }
            let next_node = ListNode::new(t % 10);
            t = t / 10;
            cur.as_mut().unwrap().next = Some(Box::new(next_node));
            cur = &mut cur.as_mut().unwrap().next;
        }

        head.unwrap().next
    }
}
```

