---
layout: post
title: "LC-Word-Search"
category: programming
---
79. Word Search
Given a 2D board and a word, find if the word exists in the grid.

The word can be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once.

Example:

board =
[
  ['A','B','C','E'],
  ['S','F','C','S'],
  ['A','D','E','E']
]

Given word = "ABCCED", return true.
Given word = "SEE", return true.
Given word = "ABCB", return false.

```
class Solution {
    public boolean exist(char[][] board, String word) {
        //check param, board is not empty, at least one
        //word is not empty
        int[][] boardUsed = new int[board.length][board[0].length];
        int[] origin = new int[2];
        int start = 0;
        int end = word.length() -1;
        return exist2(board, boardUsed, origin, word, start, end);
    }

    public int[][] arrCopy(int[][] src){
        int[][] ret = new int[src.length][];
        for(int i=0;i<src.length;i++){
            ret[i] = src[i].clone();
        }
        return ret;
    }


    public boolean exist2(char[][] board, int[][] boardUsed, int[] origin, String word, int start, int end) {
        //check params
        //get the char
        char a = word.charAt(start);
        //start from origin
        //check origin if first
        if(start == 0){ //first time
            for(int line =0;line<board.length;line++){
                for(int col =0;col<board[line].length;col++){
                    if(a == board[line][col]){
                        if(start + 1 <= end) {
                            int[][] boardUsed2 = arrCopy(boardUsed);
                            boardUsed2[line][col] = 1;
                            int[] origin2 = new int[] {line, col};
                            boolean suc = exist2(board, boardUsed2, origin2, word, start + 1, end);
                            if (suc) {
                                return true;
                            }
                        }else{
                            return true;
                        }
                    }
                }
            }
            return false;
        }
        //loop neib of origin, board
        int line = origin[0];
        int col = origin[1];
        int[][] neib = new int[][]{{line,col-1},{line,col+1},{line-1,col},{line+1, col}}; //left,right,top,bottom
        for(int i=0;i<neib.length;i++){
            int line2 = neib[i][0];
            int col2 = neib[i][1];
            if(line2 >=0 && line2 < board.length && col2 >= 0 && col2 < board[0].length){ //not cross bound
                //check used
                if(boardUsed[line2][col2] == 0){
                    //check if match char
                    if(a == board[line2][col2]){
                        //start ite
                        //check can
                        if(start + 1 <= end) {
                            int[][] boardUsed2 = arrCopy(boardUsed);
                            boardUsed2[line2][col2] = 1;
                            int[] origin2 = new int[]{line2, col2};
                            boolean suc = exist2(board, boardUsed2, origin2, word, start+1, end);
                            if(suc){
                                return true;
                            }
                        }else{
                            return true;
                        }
                    }
                }
            }
        }
        return false;
    }
}
```
