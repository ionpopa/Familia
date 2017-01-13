#include <fstream>
#include <iostream>
#include<cstdlib>
#include<cstdio>
#include <cassert>
using namespace std;

ifstream f("familie.in");
ofstream g("familie.out");

struct DLList;

struct Person
{
    string name;
    char gender;
    Person *left = NULL, *right = NULL, *mother = NULL, *father = NULL;
    DLList *children = NULL;
};

void SwapPersonPointer(Person *&p, Person *q)
{
    Person *aux = p;
    p = q;
    q = aux;
}

struct DLList
{
    struct Node
    {
        Person *person;
        Node *prev = NULL, *next = NULL;
    };

    Node *first = NULL, *last = NULL;

    void Add(Person *x)
    {
        Node *p;
        p = new Node;
        p->person = x;
        p->prev = last;
        if(!first) first = p;
        else last->next = p;
        last = p;
    }

    void Print()
    {
        Node *p;
        p = first;
        while(p)
        {
            g << p->person->name << " ";
            p = p->next;
        }
    }

    Person* Search(string name)
    {
        Person *pI;
        Node *p;
        p = first;
        while(p)
            if(p->person->name == name)
                return p->person;
            else p = p->next;

        return NULL;
    }

    bool IsEmpty()
    {
        return first == NULL;
    }
};

struct FamilyTrees
{
    Person *root = NULL;

    void Insert(Person *x)
    {
        Person *pb = NULL, *p = root;
        bool rD = true;
        while(p)
        {
            pb = p;
            if(x->name >= p->name) p = p->right, rD = true;
            else p = p->left, rD = false;
        }

        if(pb)
            if(rD) pb->right = x;
            else pb->left = x;
        else root = x;
    }

    Person* Search(string name)
    {
        Person *p = root;
        while(p && p->name != name)
            if(name >= p->name) p = p->right;
            else p = p->left;
        return p;
    }

    void InOrder()
    {
        InOrderHelper(root);
    }
    void InOrderHelper(Person *p)
    {
        if(p)
        {
            InOrderHelper(p->left);
            g << p->name << " ";
            InOrderHelper(p->right);
        }
    }

    bool AreHusbands(Person *p, Person *q,bool printMsg = true)
    {
        if(!p || !q) return false;
        if(p->children)
        {
            if(p->children->first->person->mother == q &&p->gender!=q->gender)
            {
                if(printMsg)
                    g<<p->name<<" si "<<q->name<<" sunt sot si sotie";
                return true;
            }
            else if(p->children->first->person->father == q &&p->gender!=q->gender)
            {
                if(printMsg)
                    g<<p->name<<" si "<<q->name<<" sunt sot si sotie";
                return true;
            }

        }

        return false;
    }

    bool Siblings(Person *p, Person *q, bool printMsg=true)
    {
        if(!p || !q) return true;
        if(p->father ||q->mother)
            if(p->mother == q->mother || p->father==q->father)
            {
                if(printMsg)
                {
                    g << p->name << " ";
                    if(p->gender == 'B') g << "frate ";
                    else g << "sora ";
                    g << "cu " << q->name;
                }
                return true;
            }
        return false;
    }
    bool IsParentOrChild(Person *p, Person *q, bool printMsg = true)
    {
        if(!p || !q) return false;
        if(q->father == p)
        {
            if(printMsg) g << p->name << " este tata pentru " << q->name;
            return  true;
        }
        else if(q->mother == p)
        {
            if(printMsg)  g << p->name << " este mama pentru " << q->name;
            return  true;
        }
        else if(p->mother == q || p->father == q)
        {
            if(printMsg)
            {
                g << p->name << " este";
                if(p->gender == 'B') g << " fiul";
                else g << " fica";
                g << " lui " << q->name;
            }
            return true;
        }
        return false;
    }
    bool Cousins(Person *p, Person *q)
    {
        if(!p || !q) return false;
        bool ok=false;
        if(p->father && q->mother)
        {
            if(Siblings(p->mother,q->father,false)) ok=true;
        }
        if(p->father&&q->father)
        {
            if(Siblings(p->father,q->father,false)) ok=true;
        }

        if(p->mother&&q->father)
        {
            if(Siblings(p->mother,q->mother,false)) ok=true;

        }
        if(p->mother&&q->mother)
        {
            if(Siblings(p->mother,q->mother,false)) ok=true;

        }
        if(ok)
        {
            g<<p->name<<" este";
            if(p->gender=='B') g<<" verisor";
            else g<<" verisoare";
            g<<" cu "<<q->name;
        }
    }
    bool IsGrandParentOrChild(Person *p, Person *q, bool printMsg = true)
    {
        if(!p || !q) return false;

        bool ok = false;
        if(IsParentOrChild(p,q->father,false))
        {
            if(q->gender=='B'&& q->father!=p->father)
            {
                if(printMsg)
                    g << p->name << " este bunic pentru " << q->name;
                ok = true;
            }
            else if( p->gender=='F'&& q->father!=p->father)
            {
                if(printMsg)
                    g << p->name << " este bunica pentru " << q->name;
                ok = true;
            }
        }
        else if(IsParentOrChild(p->father,q,false))
        {
            if(printMsg)
            {
                g << p->name << " este";
                if(p->gender == 'B') g << " nepotul";
                else g << " nepoata";
                g << " lui " << q->name;
                ok = true;
            }
        }
        return ok;
    }
    bool IsGrandGrandParentOrChild(Person *p, Person *q)
    {
        if(!p || !q) return false;
        bool ok = false;

        if(IsGrandParentOrChild(q->father, p, false))
        {
            g << p->name << " este strabunic pentru " << q->name;
            ok = true;
        }
        else if(IsGrandParentOrChild(q->mother, p, false))
        {
            g << p->name << " este strabunica pentru " << q->name;
            ok = true;
        }

        else if(IsGrandParentOrChild(p->father,q,false))
        {
            g << p->name << " este";
            if(p->gender == 'B') g << " stra-nepotul";
            else g << " stra-nepoata";
            g << " lui " << q->name;
            ok = true;
        }

        return ok;
    }

    bool BrotherOrSisterInLaw(Person *p, Person *q)
    {
        if(!p || !q) return true;
        if(Siblings(p,q,false)) return false;
        bool ok=false;
        Person *r;
        if(p->father)
        {
            r=p->father->children->last->person;
            if(Siblings(p,r,false))
                if(AreHusbands(r,q,false))
                    ok=true;
        }
        else if(q->father)
        {
            r=q->father->children->last->person;
            if(Siblings(q,r,false))
                if(AreHusbands(r,p,false))
                    ok=true;
        }
        else
            ok=true;
        if(ok)
        {
            g<<p->name<<" este cumnat";
            if(p->gender=='F') g<<"a";
            g<<" cu "<<q->name;
        }
        return ok;
    }

    void Relative(string xName, string yName)
    {
        Person *p, *q;
        p = Search(xName);
        q = Search(yName);
        if(AreHusbands(p, q));
        else if(Siblings(p, q));
        else if(IsParentOrChild(p, q));
        else if(IsGrandParentOrChild(p, q));
        else if(Cousins(p,q));
        else if(IsGrandGrandParentOrChild(p, q));
        else if(BrotherOrSisterInLaw(p,q));
        g << "\n";
    }
};

int main()
{
    FamilyTrees tree;
    string s, u;
    char gender;
    int n, i, j, t, m, q, k, c, l;

    f >> n;
    Person **person = new Person*[n + 1];

    for(i = 1; i <= n; ++i)
    {
        f >> j >> s >> u >> gender;
        person[j] = new Person;
        person[j]->name = s + " " + u;
        person[j]->gender = gender;
    }

    f >> l;
    for(i = 1; i <= l; ++i)
    {
        f >> t >> m >> k;
        person[t]->children = new DLList;
        person[m]->children = person[t]->children;
        /// verificare input
        if(person[t]->gender == 'B' && person[m]->gender == 'F');
        else cout << t << " " << m << "\n";

        for(j = 0; j < k; ++j)
        {
            f >> c;
            person[c]->father = person[t];
            person[c]->mother = person[m];
            person[t]->children->Add(person[c]);
        }
    }

    for(i = 1; i <= n; ++i)
        tree.Insert(person[i]);

    f >> q;
    for(k = 0; k < q; ++k)
    {
        f >> i >> j;
        tree.Relative(person[i]->name, person[j]->name);
    }

    f.close();
    g.close();
    return 0;
}

