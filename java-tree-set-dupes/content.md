### Important things to consider when writing Comparators in Java

So, I was solving this Merge Intervals [problem](https://leetcode.com/problems/merge-intervals/description/) from LeetCode and here was my first version of Comparator 
```
        Set<Integer[]> jobSet = new TreeSet<>((Integer[] a, Integer[] b) -> {
            return a[0] - b[0];
        });
```
which failed for input `int[][] intervals = {{1,4},{1,5}};`
then I modified it to following to account for the case when the start times of the job are same, but end times are different to be treated unique
```
        Set<Integer[]> jobSet = new TreeSet<>((Integer[] a, Integer[] b) -> {
            if(a[0] != b[0])
                return a[0] - b[0];
            return a[1] - b[1];
        });
```
which solved that case, but later failed for the input `int[][] intervals = {{314,315},{314,323}};`
and it is due to the fact that I was comparing the references instead of the underlying values. One possible fix for which is
```
        Set<Integer[]> jobSet = new TreeSet<>((Integer[] a, Integer[] b) -> {
            if(a[0].intValue() != b[0].intValue())
                return a[0] - b[0];
            return a[1] - b[1];
        });
```

#### Full Code for reference
```
import java.util.Set;
import java.util.TreeSet;
import java.util.List;
import java.util.ArrayList;

class MergeIntervals {
    public static void main(String[] args) {
        // int[][] intervals = {{1,3},{2,6},{8,10},{15,18}};
        // int[][] intervals = {{1,4},{4,5}};
        // int[][] intervals = {{1,4},{0,0}};
        // int[][] intervals = {{1,4},{1,4}};
        // int[][] intervals = {{1,4},{1,5}};
        // int[][] intervals = {{1,4},{0,2},{3,5}};
        // int[][] intervals = {{1,3},{0,2},{2,3},{4,6},{4,5},{5,5},{0,2},{3,3}};
        // int[][] intervals = {{2,3},{4,5},{6,7},{8,9},{1,10}};
        // int[][] intervals = {{362,367},{314,315},{133,138},{434,443},{202,203},{144,145},{229,235},{205,212},{314,323},{128,129},{413,414},{342,345},{43,49},{333,342},{173,178},{386,391},{131,133},{157,163},{187,190},{186,186},{17,19},{63,69},{70,79},{386,391},{98,102},{236,239},{195,195},{338,338},{169,170},{151,153},{409,416},{377,377},{90,96},{156,165},{182,186},{371,372},{228,233},{297,306},{56,61},{184,190},{401,403},{221,228},{203,212},{39,43},{83,84},{66,68},{80,83},{32,32},{182,182},{300,306},{235,238},{267,272},{458,464},{114,120},{452,452},{372,375},{275,280},{302,302},{5,9},{54,62},{237,237},{432,439},{415,421},{340,347},{356,358},{165,168},{15,17},{259,265},{201,204},{192,197},{376,383},{210,211},{362,367},{481,488},{59,64},{307,315},{155,164},{465,467},{55,60},{20,24},{297,304},{207,210},{322,328},{139,142},{192,195},{28,36},{100,108},{71,76},{103,105},{34,38},{439,441},{162,168},{433,433},{368,369},{137,137},{105,112},{278,280},{452,452},{131,132},{475,480},{126,129},{95,104},{93,99},{394,403},{70,78}};
        int[][] intervals = {{314,315},{314,323},{307,315},{322,328}};
        int[][] result = merge(intervals);
        for(int i = 0; i < result.length; i++) {
            // System.out.println("[" + result[i][0] + "," + result[i][1] + "]");
        }
    }
    
    public static int[][] merge(int[][] intervals) {
        Set<Integer[]> jobSet = new TreeSet<>((Integer[] a, Integer[] b) -> {
            System.out.println("comparing a " + a + " and b " + b + " whose values are a[0] = " + a[0].hashCode() + ", a[1] = " + a[1].hashCode() + " and b[0] = " + b[0].hashCode() + ", b[1] = " + b[1].hashCode());
            if(a[0].intValue() != b[0].intValue())
                return a[0] - b[0];
            return a[1] - b[1];
        });
        Set<Job> jSet = new TreeSet<>((Job j1, Job j2) -> {
            if(j1.start != j2.start)
                return j1.start - j2.start;
            return j1.end - j2.end;
        });
        for(int i = 0; i < intervals.length; i++) {
            jobSet.add(new Integer[]{intervals[i][0], intervals[i][1]});
            jSet.add(new Job(intervals[i][0], intervals[i][1]));
        }
        System.out.println(intervals.length);
        System.out.println(jobSet.size());
        for(Integer[] currentJob: jobSet) {
            System.out.println("[" + currentJob[0] + "," + currentJob[1] + "]");
        }
        System.out.println(jobSet);
        System.out.println(jSet);
        int[][] result = intervals;
        List<Integer[]> jobs = new ArrayList<>();
        for(Integer[] currentJob: jobSet) {
            Integer[] job = null;
            for(int j = 0; j < jobs.size(); j++) {
                job = jobs.get(j);
                if(currentJob[1] >= job[0] && currentJob[0] <= job[1]) {
                    break;
                }
                job = null;
            }
            if(job == null) {
                jobs.add(new Integer[]{currentJob[0], currentJob[1]});
            } else {
                if(job[0] != currentJob[0] || job[1] != currentJob[1]) {
                    job[0] = Math.min(job[0], currentJob[0]);
                    job[1] = Math.max(job[1], currentJob[1]);
                }
            }
        }
        result = new int[jobs.size()][2];
        for(int i = 0; i < jobs.size(); i++) {
            result[i][0] = jobs.get(i)[0];
            result[i][1] = jobs.get(i)[1];
        }
        return result;
    }
    static class Job {
        int start;
        int end;
        public Job(int start, int end) {
            this.start = start;
            this.end = end;
        }
        // public String toString() {
        //     return "[" + start + "," + end + "]";
        // }
    }
}

```
