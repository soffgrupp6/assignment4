# Report for assignment 4

## Project

Name: williamfiset/Algorithms

URL: https://github.com/williamfiset/Algorithms/

This repository has a variety of algorithms and data structures implemented with Java.

## Onboarding experience

We went with the same project as the previous assignment so there was no new onboarding. In the last assignment our onboarding went well as the only requirements for running tests were gradle and the installation was trivial. Building and running the tests from gradle was only the matter of running `gradle test` in the terminal.

## Effort spent

For each team member, how much time was spent:

| Topic                                 | Anja | Christian | Daniel | Timmy |
|---------------------------------------|-----:|----------:|-------:|------:|
| Preliminary discussions               |   2  |         2 |      2 |     2 |
| Discussions within parts of the group |   0  |         3 |      0 |     0 |
| Reading documentation                 |      |         0 |        |       |
| Analyzing code/output                 |      |         6 |        |       |
| Writing documentation                 |      |         3 |        |       |
| Writing code                          |      |         4 |        |       |
| Running code                          |      |         1 |        |       |


## More on refactoring

| Topic                                | Effort | How often used | Outcome |
|--------------------------------------|-------:|---------------:|--------:|
| Rename field (Christian)             |    Low |          Often |  Very much more understandable code with better naming practices |
| Extract method                       | Medium  |               |       - |
| Replace conditional with polymorphism |        |               |       - |


## Overview of issue(s) and work done.

Title: Refactor and add tests for Skip List implementation.
URL: https://github.com/williamfiset/Algorithms/issues/74

The issue is about refactoring an implementation of a Skip List and adding of a complete new test suite. The scope of the issue is very local to the file with the Skip List implementation and the test file to be created.

When we started looking at this issue, we intended to restructure and refactor the code, and write some tests for the implementation as specified by the issue. However, after analyzing the code it soon became clear that the code needed not only refactoring, but actually rewriting. Many methods were not only messy but also buggy or simply wrong. Some parts had unresonable implementations of methods. Tests were alse completely missing, probably because the code did not work.

The work needed to resolve this issue was thus much more extensive than we first expected.

## Requirements affected by functionality being refactored

#### Optional (point 3):

This is how we traced the tests to the requirements...


## Code changes

### Patch


```diff
diff --git a/src/main/java/com/williamfiset/algorithms/datastructures/skiplist/SkipList.java b/src/main/java/com/williamfiset/algorithms/datastructures/skiplist/SkipList.java
index 76c873e..5354086 100644
--- a/src/main/java/com/williamfiset/algorithms/datastructures/skiplist/SkipList.java
+++ b/src/main/java/com/williamfiset/algorithms/datastructures/skiplist/SkipList.java
@@ -1,8 +1,8 @@
 /**
  * SkipList is a data structure that is useful for dealing with dynamic sorted data. In particular
  * it gives O(log(n)) average complexity of insertion, removal, and find operations. This
- * implementation has been augmented with a method for determining the rank of an element in the
- * SkipList. Finding the rank of an element is also O(log(n)) on average. The complexities are
+ * implementation has been augmented with a method for determining the index of an element in the
+ * SkipList. Finding the index of an element is also O(log(n)) on average. The complexities are
  * average complexities since this algorithm is dependent on randomisation to achieve nice balanced
  * properties. To make this efficient, instantiate the SkipList with a height equal to, or just
  * greater than, log(n) where n is the number of elements that will be in the list. On average, this
@@ -16,150 +16,186 @@ package com.williamfiset.algorithms.datastructures.skiplist;
 import java.util.Random;

 class SkipList {
-  Random rand = new Random(0);
-  int height;
-  Node head;
-  Node tail;
-  // Initialise with the height of the SkipList and Keys smaller and larger
-  // than all other keys that will be inserted in the SkipList
-  public SkipList(int height, Key minKey, Key maxKey, int h) {
+  private Random rand = new Random();
+  private int height;
+  private Node head;
+  private Node tail;
+
+  /**
+   * This function creates a new skip list with the specified height and minimum and maximum values.
+   */
+  public SkipList(int height, int minValue, int maxValue) {
+    // Height goes from 0 to height-1
     this.height = height;
-    head = new Node(minKey);
-    tail = new Node(maxKey);
+    head = new Node(minValue);
+    tail = new Node(maxValue);
     Node currLeft = head;
     Node currRight = tail;
-    for (int i = 0; i <= h; i++) {
-      currLeft.right = currRight;
-      currRight.left = currLeft;
-      currLeft.down = new Node(currLeft.k);
+    // Setup links between all levels of head and tail nodes
+    for (int i = 1; i < height; i++) {
+      setHeadTail(height - i, currLeft, currRight);
+      // Create and setup down node
+      currLeft.down = new Node(currLeft.value);
       currLeft.down.up = currLeft;
-      currRight.down = new Node(currRight.k);
+      currRight.down = new Node(currRight.value);
       currRight.down.up = currRight;
-      currLeft.leftDist = 0;
-      currRight.leftDist = 1;
-      currLeft.height = height - i;
-      currRight.height = height - i;
       currLeft = currLeft.down;
       currRight = currRight.down;
     }
-    currLeft.up.down = null;
-    currRight.up.down = null;
+    // Set last level
+    setHeadTail(0, currLeft, currRight);
   }

-  public void insert(Node n2) {
-    int r = rand.nextInt();
-    if (r < 0) r *= -1;
-    r %= (1 << (height - 1));
-    int nodeHeight = Integer.numberOfLeadingZeros(r) - (32 - (height - 1));
-    head.find(n2).insert(n2, null, nodeHeight, 1);
+  private static void setHeadTail(int height, Node nLeft, Node nRight) {
+    nLeft.right = nRight;
+    nRight.left = nLeft;
+    nLeft.index = 0;
+    nRight.index = 1;
+    nLeft.height = height;
+    nRight.height = height;
   }

-  public void remove(Node n2) {
-    head.find(n2).remove(n2);
+  public int size() {
+    return this.tail.index + 1;
   }

-  public int getRank(Node n2) {
-    Node curr = head.find(n2);
-    if (curr.compareTo(n2) != 0) return -1;
-    int distSum = 0;
-    while (curr.left != null) {
-      while (curr.up != null) {
-        curr = curr.up;
-      }
-      distSum += curr.leftDist;
-      curr = curr.left;
-    }
-    return distSum;
+  /** Return true if the number is in the list. False otherwise. */
+  public boolean find(int num) {
+    return (search(num).value == num);
   }

-  class Node implements Comparable<Node> {
-    Node left;
-    Node right;
-    Node up;
-    Node down;
-    int height;
-    int leftDist;
-    Key k;
+  // Search for closest node by number
+  private Node search(int num) {
+    return search(num, this.head);
+  }

-    public Node(Key kk) {
-      k = kk;
+  // Helper method for search
+  private Node search(int num, Node node) {
+    // Check if the next right node is still less than num
+    if (node.compareTo(num) < 0) {
+      if (node.down != null) return search(num, node.down);
+      else if (node.right != null && node.right.compareTo(num) <= 0) return search(num, node.right);
     }
+    return node;
+  }

-    public int compareTo(Node n2) {
-      return k.compareTo(n2.k);
+  /** Inserts the number into the list. */
+  public boolean insert(int num) {
+    if (num < head.value || num > tail.value) return false;
+    Node node = search(num);
+    if (node.value == num) return false;
+    int nodeHeight = 0;
+    // 0.5 probability of having height 1, 0.25 for height 2
+    while (rand.nextBoolean() && nodeHeight < (height - 1)) {
+      nodeHeight++;
     }
+    insert(node, new Node(num), null, nodeHeight, node.index + 1);
+    return true;
+  }

-    public Node find(Node f) {
-      if (f.compareTo(right) >= 0) {
-        return right.find(f);
-      } else if (down != null) {
-        return down.find(f);
+  /** Helper method for insert */
+  private void insert(Node startNode, Node insertNode, Node lower, int insertHeight, int distance) {
+    if (startNode.height <= insertHeight) {
+      insertNode.left = startNode;
+      insertNode.right = startNode.right;
+      // If not at lowest level
+      if (lower != null) {
+        lower.up = insertNode;
+        insertNode.down = lower;
       }
-      return this;
-    }
-
-    public void insert(Node n2, Node lower, int insertHeight, int distance) {
-      if (height <= insertHeight) {
-        n2.left = this;
-        n2.right = right;
-        n2.down = lower;
-        right.left = n2;
-        right = n2;
-        if (lower != null) lower.up = n2;
-        n2.height = height;
-        n2.leftDist = distance;
-        n2.right.leftDist -= n2.leftDist - 1;
-        Node curr = this;
-        while (curr.up == null) {
-          distance += curr.leftDist;
-          curr = curr.left;
-        }
+      // If at lowest level, update ranks of following
+      else if (startNode.height == 0) {
+        increaseRank(insertNode.right);
+      }
+      startNode.right.left = insertNode;
+      startNode.right = insertNode;
+      insertNode.height = startNode.height;
+      insertNode.index = distance;
+      Node curr = startNode;
+      while (curr.up == null && curr.left != null) {
+        curr = curr.left;
+      }
+      if (curr.up != null) {
         curr = curr.up;
-        curr.insert(new Node(n2.k), n2, insertHeight, distance);
-      } else {
-        Node curr = this;
-        curr.right.leftDist++;
-        while (curr.left != null || curr.up != null) {
-          while (curr.up == null) {
-            curr = curr.left;
-          }
-          curr = curr.up;
-          curr.right.leftDist++;
-        }
+        insert(curr, new Node(insertNode.value), insertNode, insertHeight, distance);
       }
     }
+  }

-    public void remove(Node n2) {
-      Node curr = this;
-      if (curr.compareTo(n2) != 0) return;
-      while (curr.up != null) {
-        curr.left.right = curr.right;
-        curr.right.left = curr.left;
-        curr.right.leftDist += curr.leftDist - 1;
-        curr = curr.up;
-      }
-      curr.left.right = curr.right;
-      curr.right.left = curr.left;
-      curr.right.leftDist += curr.leftDist - 1;
-      while (curr.left != null || curr.up != null) {
-        while (curr.up == null) {
-          curr = curr.left;
-        }
-        curr = curr.up;
-        curr.right.leftDist--;
+  /**
+   * Remove a the number from the list with the given value. Returns true if value was successfully
+   * removed. False otherwise.
+   */
+  public boolean remove(int num) {
+    // Get the node to remove
+    Node node = search(num);
+    if (node.value != num) return false;
+    // Re-number all nodes to the right
+    decreaseRank(node.right);
+
+    // Connect the left and right nodes to each other
+    while (node.up != null) {
+      node.left.right = node.right;
+      node.right.left = node.left;
+      node = node.up;
+    }
+    node.left.right = node.right;
+    node.right.left = node.left;
+
+    return true;
+  }
+
+  // Decrease the index for all nodes to the right of the start node
+  private void decreaseRank(Node startNode) {
+    modifyRank(startNode, -1);
+  }
+
+  // Increase the index for all nodes to the right of the start node
+  private void increaseRank(Node startNode) {
+    modifyRank(startNode, 1);
+  }
+
+  // Modify the index for all nodes to the right of the start node
+  private void modifyRank(Node startNode, int change) {
+    Node node = startNode;
+    // Check all nodes to the right
+    while (startNode != null) {
+      node = startNode;
+      // Check all nodes upwards
+      while (node.up != null) {
+        node.index += change;
+        node = node.up;
       }
+      node.index += change;
+      startNode = startNode.right;
     }
   }

-  class Key implements Comparable<Key> {
-    int k;
+  /** Returns the index of the number in the list or -1 if not found. */
+  public int getIndex(int num) {
+    Node node = search(num);
+    return num == node.value ? node.index : -1;
+  }

-    public Key(int kk) {
-      k = kk;
+  class Node implements Comparable<Node> {
+    Node left;
+    Node right;
+    Node up;
+    Node down;
+    int height;
+    int index;
+    int value;
+
+    public Node(int value) {
+      this.value = value;
+    }
+
+    public int compareTo(Node n2) {
+      return Integer.compare(this.value, n2.value);
     }

-    public int compareTo(Key k2) {
-      return k - k2.k;
+    public int compareTo(int num) {
+      return Integer.compare(this.value, num);
     }
   }
 }

```

#### Optional (point 4):

The patch is clean and formatted according to the the specified code standard of the repository. We also took effort in making the tests formatted like other tests in the repository.

#### Optional (point 5):

The patch passes all automated checks and is considered for acceptance.

## Test results

The SkipList class did not have any tests prior to our implementation. Here is the output from the test suite:

<div id="tab0" class="tab selected">

<table>
<thead>
<tr>
<th>Test</th>
<th>Duration</th>
<th>Result</th>
</tr>
</thead>
<tbody>
<tr>
<td class="success">testDuplicate</td>
<td class="success">0.003s</td>
<td class="success">passed</td>
</tr>
<tr>
<td class="success">testFind</td>
<td class="success">0.034s</td>
<td class="success">passed</td>
</tr>
<tr>
<td class="success">testIndex</td>
<td class="success">0.021s</td>
<td class="success">passed</td>
</tr>
<tr>
<td class="success">testIndexWithNonExistingValue</td>
<td class="success">0.002s</td>
<td class="success">passed</td>
</tr>
<tr>
<td class="success">testInsertGreaterThanMaxValue</td>
<td class="success">0.003s</td>
<td class="success">passed</td>
</tr>
<tr>
<td class="success">testInsertLesserThanMinValue</td>
<td class="success">0s</td>
<td class="success">passed</td>
</tr>
<tr>
<td class="success">testNegativeNumbers</td>
<td class="success">0.017s</td>
<td class="success">passed</td>
</tr>
<tr>
<td class="success">testRemoveNonExisting</td>
<td class="success">0.004s</td>
<td class="success">passed</td>
</tr>
<tr>
<td class="success">testSimpleFunctionality</td>
<td class="success">0.005s</td>
<td class="success">passed</td>
</tr>
<tr>
<td class="success">testSize</td>
<td class="success">0.009s</td>
<td class="success">passed</td>
</tr>
</tbody></table>
</div>

### Test patch

The test file can be viewed [here](https://github.com/soffgrupp6/Algorithms/blob/issue/74/src/test/java/com/williamfiset/algorithms/datastructures/skiplist/SkipListTest.java).

## UML class diagram and its description

### Key changes/classes affected

Optional (point 1): Architectural overview.

Optional (point 2): relation to design pattern(s).

## Overall experience

What are your main take-aways from this project? What did you learn?


How did you grow as a team, using the Essence standard to evaluate yourself?

Optional (point 6): How would you put your work in context with best software engineering practice?

Optional (point 7): Is there something special you want to mention here?
