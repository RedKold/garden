```cpp
typedef struct Node{
    Node* son[26]{};
    bool end = false;
} Node;
class Trie {
   Node* root = new Node(); 
   int find(string word){
     Node* cur = root;
    for(auto c:word){
        int i = c-'a';
        if(cur->son[i]==nullptr){
            // we don't have this son;
            return 0;
        }
        cur=cur->son[i];
    }
    return cur->end? 2:1;
   }
public:
    
    void insert(string word) {
        Node* cur = root;
        for(auto c:word){
            int i = c-'a';
            if(cur->son[i] == nullptr){
                cur->son[i] = new Node();
            }
            cur = cur->son[i];
        }
        cur->end=true;
    }
    
    bool search(string word) {
        return find(word)==2;
    }
    
    bool startsWith(string prefix) {
        return find(prefix)>0;
    }
};

/**
 * Your Trie object will be instantiated and called as such:
 * Trie* obj = new Trie();
 * obj->insert(word);
 * bool param_2 = obj->search(word);
 * bool param_3 = obj->startsWith(prefix);
 *
```

