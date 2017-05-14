# DecisionTree
implementation of DecisionTree(CART)  

cuteboydot@gmail.com  

reference : http://edoc.hu-berlin.de/master/timofeev-roman-2004-12-20/PDF/timofeev.pdf  

- entropy, gini impurity and inforamtion gain  
entropy impurity = -( Sum(Pi * log2(Pi)) ), i = class0,1,..,   Pi = cnt(node_class_i_data)/cnt(node_all_data)  
gini impurity = 1 - Sum(Pi * Pi)  
information gain(IG) = impurity(node) - (cnt(node_left_data)/cnt(node_all_data) * impurity(node_left) + cnt(node_right_data)/cnt(node_all_data) * impurity(node_right))

- example  
<br>
<img height="400" src="https://github.com/cuteboydot/DecisionTree/blob/master/img/dataset.png" />
</br>

- test
<br>
<img height="500" src="https://github.com/cuteboydot/DecisionTree/blob/master/img/resultset.png" />
</br>

<br>
<img src="https://github.com/cuteboydot/DecisionTree/blob/master/img/result.png" />
</br>

- code  
```java
import java.util.*;

/**
 * Created by cuteboydot
 */

class DTData {
    int id;
    double[] feat = null;
    int label;      // 0:male, 1:female
}

class DTSplit {
    int feat;
    double value;
}

class DTNode {
    int depth;
    int maxdepth;
    int direction;   // 0:root, 1:left, 2:right
    String path;
    int cnt;        // node count
    double impurity;
    DTSplit splitrule = null;
    DTNode childLeft = null;
    DTNode childRight = null;
    ArrayList<DTData> data = null;
}

public class Main {
    static final int SIZE_FEAT = 2;
    static final int SIZE_LABEL = 2;
    static final int SIZE_RECORD = 12;
    static final int SIZE_TEST = 2;

    public static void main(String[] args) {
        System.out.println("hello world~!");

        DTData[] data = new DTData[SIZE_RECORD] ;
        DTData[] test = new DTData[SIZE_TEST] ;

        data[0] = new DTData();
        data[0].id = 0;
        data[0].feat = new double[SIZE_FEAT];
        data[0].feat[0] = 1;
        data[0].feat[1] = 4;
        data[0].label = 0;

        data[1] = new DTData();
        data[1].id = 1;
        data[1].feat = new double[SIZE_FEAT];
        data[1].feat[0] = 3;
        data[1].feat[1] = 3;
        data[1].label = 0;

        data[2] = new DTData();
        data[2].id = 2;
        data[2].feat = new double[SIZE_FEAT];
        data[2].feat[0] = 4;
        data[2].feat[1] = 5;
        data[2].label = 0;

        data[3] = new DTData();
        data[3].id = 3;
        data[3].feat = new double[SIZE_FEAT];
        data[3].feat[0] = 5;
        data[3].feat[1] = 6;
        data[3].label = 0;

        data[4] = new DTData();
        data[4].id = 4;
        data[4].feat = new double[SIZE_FEAT];
        data[4].feat[0] = 6;
        data[4].feat[1] = 2;
        data[4].label = 0;

        data[5] = new DTData();
        data[5].id = 5;
        data[5].feat = new double[SIZE_FEAT];
        data[5].feat[0] = 3;
        data[5].feat[1] = 4;
        data[5].label = 1;

        data[6] = new DTData();
        data[6].id = 6;
        data[6].feat = new double[SIZE_FEAT];
        data[6].feat[0] = 4;
        data[6].feat[1] = 1;
        data[6].label = 1;

        data[7] = new DTData();
        data[7].id = 7;
        data[7].feat = new double[SIZE_FEAT];
        data[7].feat[0] = 5;
        data[7].feat[1] = 3;
        data[7].label = 1;

        data[8] = new DTData();
        data[8].id = 8;
        data[8].feat = new double[SIZE_FEAT];
        data[8].feat[0] = 7;
        data[8].feat[1] = 5;
        data[8].label = 1;

        data[9] = new DTData();
        data[9].id = 9;
        data[9].feat = new double[SIZE_FEAT];
        data[9].feat[0] = 8;
        data[9].feat[1] = 3;
        data[9].label = 1;

        data[10] = new DTData();
        data[10].id = 10;
        data[10].feat = new double[SIZE_FEAT];
        data[10].feat[0] = 2;
        data[10].feat[1] = 1;
        data[10].label = 0;

        data[11] = new DTData();
        data[11].id = 11;
        data[11].feat = new double[SIZE_FEAT];
        data[11].feat[0] = 6;
        data[11].feat[1] = 5;
        data[11].label = 1;

        test[0] = new DTData();
        test[0].id = 0;
        test[0].feat = new double[SIZE_FEAT];
        test[0].feat[0] = 2;
        test[0].feat[1] = 5;
        test[0].label = -1;     // real answer = 0

        test[1] = new DTData();
        test[1].id = 1;
        test[1].feat = new double[SIZE_FEAT];
        test[1].feat[0] = 6;
        test[1].feat[1] = 4;
        test[1].label = -1;     // real answer = 1

        DTNode trainTree = new DTNode();
        trainTree.data = new ArrayList<DTData>();
        for(int a=0; a<SIZE_RECORD; a++) {
            trainTree.data.add(data[a]);
        }
        trainTree.depth = 0;
        trainTree.maxdepth = 5;
        trainTree.direction = 0;
        trainTree.path = "ROOT";
        trainTree.cnt = trainTree.data.size();

        // train
        train(trainTree);
        System.out.println();

        // test
        for (int a=0; a<SIZE_TEST; a++) {
            test(trainTree, test[a]);
            System.out.println("TEST#" + test[a].id + " RESULT : " + test[a].label);
        }
    }

    static void train(DTNode tree) {

        expandTree(tree);

    }

    static void expandTree(DTNode tree) {
        tree.impurity = getImpurity(tree);

        String strDir = "";
        if(tree.direction == 0) strDir = "ROOT";
        else if(tree.direction == 1) strDir = "LEFT";
        else if(tree.direction == 2) strDir = "RIGHT";

        // terminal node
        if((tree.depth >= tree.maxdepth) || (tree.impurity == 0) || (tree.data.size() == 1)) {
            //System.out.println("DEPTH: " + tree.depth + ", DIRECTION: " + strDir + ", CNT: " + tree.cnt +
            System.out.println("PATH: " + tree.path + ", CNT: " + tree.cnt +
                    "[TERMINAL], IMPURITY: " + String.format("%.3f",tree.impurity));
            return;
        }

        // make split criteria list
        ArrayList<Double>[] splitList = new ArrayList[SIZE_FEAT];
        for (int a=0; a<SIZE_FEAT; a++) {
            splitList[a] = new ArrayList<Double>();

            for (int b=0; b<tree.data.size(); b++) {
                splitList[a].add( tree.data.get(b).feat[a] );
            }

            // sort value list
            Collections.sort(splitList[a], new Comparator<Double>() {
                @Override
                public int compare(Double o1, Double o2) {
                    return o1.compareTo(o2);
                }
            });

            // remove duplicated value
            for (int b=tree.data.size()-2; b>=0; b--) {
                Double pre = splitList[a].get(b);
                Double suc = splitList[a].get(b+1);
                if(Double.compare(pre, suc) == 0) {
                    splitList[a].remove(b + 1);
                }
            }
        }

        for (int a=0; a<SIZE_FEAT; a++) {
            // split value = (data[pre] + data[suc]) / 2
            for (int b=0; b<splitList[a].size()-1; b++) {
                double val = (splitList[a].get(b) + splitList[a].get(b+1)) / 2;
                splitList[a].set(b, val);
            }
            splitList[a].remove(splitList[a].size()-1);
        }

        // allocate tmp sub tree by criteria rule
        // find maximum delta gain of sub trees
        int maxSplitFeat = 0;
        double maxSplitValue = 0;
        double maxSplitGain = 0;
        for (int a=0; a<SIZE_FEAT; a++) {
            for (int b=0; b<splitList[a].size(); b++) {
                double val = splitList[a].get(b);
                DTNode tmpLeftTree = new DTNode();
                DTNode tmpRightTree = new DTNode();

                for( int c=0; c<tree.data.size(); c++) {
                    if(tree.data.get(c).feat[a] <= val) {
                        if(tmpLeftTree.data == null) {
                            tmpLeftTree.data = new ArrayList<DTData>();
                        }
                        tmpLeftTree.data.add(tree.data.get(c));
                    } else {
                        if(tmpRightTree.data == null) {
                            tmpRightTree.data = new ArrayList<DTData>();
                        }
                        tmpRightTree.data.add(tree.data.get(c));
                    }
                }

                double impLeft = getImpurity(tmpLeftTree);
                double impRight = getImpurity(tmpRightTree);

                double gain = tree.impurity -
                        ( ((double)tmpLeftTree.data.size()/(double)tree.data.size()) * impLeft +
                          ((double)tmpRightTree.data.size()/(double)tree.data.size()) * impRight );

                if(gain > maxSplitGain) {
                    maxSplitFeat = a;
                    maxSplitValue = val;
                    maxSplitGain = gain;
                }

            }
        }

        tree.splitrule = new DTSplit();
        tree.splitrule.feat = maxSplitFeat;
        tree.splitrule.value = maxSplitValue;

        tree.childLeft = new DTNode();
        tree.childLeft.data = new ArrayList<DTData>();
        tree.childLeft.maxdepth = tree.maxdepth;
        tree.childLeft.depth = tree.depth + 1;
        tree.childLeft.direction = 1;
        tree.childLeft.path = tree.path + " -> " + tree.childLeft.depth + "L";

        tree.childRight = new DTNode();
        tree.childRight.data = new ArrayList<DTData>();
        tree.childRight.maxdepth = tree.maxdepth;
        tree.childRight.depth = tree.depth + 1;
        tree.childRight.direction = 2;
        tree.childRight.path = tree.path + " -> " + tree.childLeft.depth + "R";

        // make child tree
        for(int a=0; a<tree.data.size(); a++) {
            DTData data = tree.data.get(a);

            if(data.feat[tree.splitrule.feat] <= tree.splitrule.value) {
                tree.childLeft.data.add(data);
            } else {
                tree.childRight.data.add(data);
            }
        }

        tree.childLeft.cnt = tree.childLeft.data.size();
        tree.childRight.cnt = tree.childRight.data.size();

        //System.out.println("DEPTH: " + tree.depth + ", DIRECTION: " + strDir + ", CNT: " + tree.cnt +
        System.out.println("PATH: " + tree.path + ", CNT: " + tree.cnt +
                "[" + tree.childLeft.cnt + "," + tree.childRight.cnt + "]" +
                ", IMPURITY: " + String.format("%.3f",tree.impurity) + ", SPLIT FEAT: " + tree.splitrule.feat +
                ", SPLIT VAL: " + String.format("%.3f",tree.splitrule.value));

        // expand subtree recursively
        if(tree.childLeft.cnt >= 1)
            expandTree(tree.childLeft);
        if(tree.childRight.cnt >= 1)
            expandTree(tree.childRight);
    }

    static void test(DTNode trainTree, DTData test) {

        test.label = propagation(trainTree, test);
    }

    static int propagation(DTNode subTree, DTData test) {
        int label = 0;
        int cntClas[] = new int[SIZE_LABEL];
        int tmpCnt = 0;

        if(subTree.data.size() == 1) {
            label = subTree.data.get(0).label;
            return label;
        }

        for (int a=0; a<subTree.data.size(); a++) {
            DTData data = subTree.data.get(a);
            cntClas[data.label]++;
        }

        for (int a=0; a<SIZE_LABEL; a++) {
            tmpCnt = cntClas[a];
            if(tmpCnt > label)
                label = a;
        }

        // case of classified cluster or terminal node
        if(subTree.impurity == 0.0 || subTree.depth >= subTree.maxdepth) {
            return label;
        }

        if(test.feat[subTree.splitrule.feat] <= subTree.splitrule.value) {
            if(subTree.childLeft != null) {
                if(subTree.childLeft.data.size() > 1) {
                    label = propagation(subTree.childLeft, test);
                }
            }
        } else {
            if(subTree.childRight != null) {
                if(subTree.childRight.data.size() > 1) {
                    label = propagation(subTree.childRight, test);
                }
            }
        }

        cntClas = null;
        return label;
    }

    static double getImpurity(DTNode node) {
        double impurity;

        if(node.data.size() == 1) {
            impurity = 0;
            return impurity;
        }

        impurity = getEntropy(node);
        //impurity = getGini(node);

        return  impurity;
    }

    static double getEntropy(DTNode node) {
        double impurity = 0;
        int cnt[] = new int [SIZE_LABEL];
        int totalcnt = node.data.size();

        for (int a=0; a<totalcnt; a++) {
            cnt[node.data.get(a).label]++;
        }

        for (int a=0; a<SIZE_LABEL; a++) {
            double prob = (double)cnt[a]/(double)totalcnt;
            double log2 = 0.0;

            if(prob == 0.0) {
                log2 = 0.0;
            } else {
                log2 = Math.log(prob)/Math.log(2);
            }

            impurity += (prob * log2);
        }
        impurity *= -1;

        return impurity;
    }

    static double getGini(DTNode node) {
        double impurity = 0;
        int cnt[] = new int [SIZE_LABEL];
        int totalcnt = node.data.size();

        for (int a=0; a<totalcnt; a++) {
            cnt[node.data.get(a).label]++;
        }

        for (int a=0; a<SIZE_LABEL; a++) {
            double prob = (double)cnt[a]/(double)totalcnt;

            impurity += (prob * prob);
        }
        impurity = 1 - impurity;

        return impurity;
    }
}
```
