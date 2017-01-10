
#include <fstream>
#include <iostream>
#include<cstdlib>
#include<cstdio>
#include <cassert>
using namespace std;

ifstream f("familie.in");
ofstream g("familie.out");

struct DLList;

struct Person {
	string name;
	char gender;
	Person *left = NULL, *right = NULL, *mother = NULL, *father = NULL;
	DLList *children = NULL;
};

void SwapPersonPointer(Person *&p, Person *q) {
	Person *aux = p; p = q; q = aux;
}

struct DLList {
	struct Node {
		Person *person;
		Node *prev = NULL, *next = NULL;
	};

	Node *first = NULL, *last = NULL;

	void Add(Person *x) {
		Node *p;
		p = new Node;
		p->person = x;
		p->prev = last;
		if(!first) first = p;
		else last->next = p;
		last = p;
	}

	void Print() {
		Node *p;
		p = first;
		while(p) {
			g << p->person->name << " ";
			p = p->next;
		}
	}

	Person* Search(string name) {
		Person *pI;
		Node *p;
		p = first;
		while(p)
			if(p->person->name == name)
				return p->person;
			else p = p->next;

		return NULL;
	}

	bool IsEmpty() {
		return first == NULL;
	}
};

struct FamilyTrees {
	Person *root = NULL;

	void Insert(Person *x) {
		Person *pb = NULL, *p = root;
		bool rD = true;
		while(p) {
			pb = p;
			if(x->name >= p->name) p = p->right, rD = true;
			else p = p->left, rD = false;
		}
		if(pb)
			if(rD) pb->right = x;
			else pb->left = x;
		else root = x;
	}

	Person* Search(string name) {
		Person *p = root;
		while(p && p->name != name)
			if(name >= p->name) p = p->right;
			else p = p->left;
		return p;
	}

	void InOrder() {
		InOrderHelper(root);
	}
	void InOrderHelper(Person *p) {
		if(p) {
			InOrderHelper(p->left);
			g << p->name << " ";
			InOrderHelper(p->right);
		}
	}

	bool AreHusbands(Person *p, Person *q) {
		if(!p || !q) return false;
		bool ok=false;
        for(int i=0;i<2;++i){
		if(p->children)
			if(p->children->first->person->mother == q)
			    if(p->name!=q->name)
            {
				g<<p->name<<" si "<<q->name<<" sunt sot si sotie";
				return true;
			}

            SwapPersonPointer(p,q);
        }

		return false;
	}

	bool Siblings(Person *p, Person *q, bool printMsg=true) {
		if(!p || !q) return false;

		if(p->mother == q->mother) {
			if(printMsg){
                g << p->name << " ";
                if(p->gender == 'B') g << "frate ";
                else g << "sora ";
                g << "cu " << q->name;
			}

			return true;
		}

		return false;
	}

	bool IsParentOrChild(Person *p, Person *q, bool printMsg = true) { 
		if(!p || !q) return false;

		bool ok = false;
		if(q->father == p) {
			if(printMsg) g << p->name << " este tata pentru " << q->name;
			ok = true;
		} else if(q->mother == p) {
			if(printMsg)  g << p->name << " este mama pentru " << q->name;
			ok = true;
		} else if(p->mother == q || p->father == q) {
			if(printMsg) {
				g << p->name << " este";
				if(p->gender == 'B') g << " fiul";
				else g << " fica";
				g << " lui " << q->name;
			}

			ok = true;
		}

		return ok;
	}


    bool Cousins(Person *p, Person *q)
    {
        if(!p || !q) return false;

        bool ok=false;
        if(p->father && q->mother)
        {
              if(Siblings(p->mother,q->father,false)!=false) ok=true;
        }
         if(p->father&&q->father)
        {
            if(Siblings(p->father,q->father,false)!=false) ok=true;
        }

         if(p->mother&&q->father)
        {
            if(Siblings(p->mother,q->mother,false)!=false) ok=true;

        }
         if(p->mother&&q->mother)
        {
            if(Siblings(p->mother,q->mother,false)!=false) ok=true;

        }


        if(ok){
            g<<p->name<<" este";
            if(p->gender=='B') g<<" verisor";
            else g<<" verisoare";
            g<<" cu "<<q->name;
        }
    }

   
	void Relative(string xName, string yName) {
		Person *p, *q;
		p = Search(xName);
		q = Search(yName);
		if(p->gender != q->gender && p->gender == 'F') SwapPersonPointer(p, q);
		if(AreHusbands(p, q));
		else if(Siblings(p, q));
		else if(IsParentOrChild(p, q));
		else if(Cousins(p,q));
		g << "\n";
	}
};

int main() {
	FamilyTrees tree;
	string s, u; char gender;
	int n, i, j, t, m, q, k, c, l;

	f >> n;
	Person **person = new Person*[n + 1];

	for(i = 1; i <= n; ++i) {
		f >> j >> s >> u >> gender;
		person[j] = new Person;
		person[j]->name = s + " " + u;
		person[j]->gender = gender;
	}

	f >> l;
	for(i = 1; i <= l; ++i) {
		f >> t >> m >> k;
		person[t]->children = new DLList;
		person[m]->children = person[t]->children;
		if(person[t]->gender == 'B' && person[m]->gender == 'F');
		else cout << t << " " << m << "\n";

		for(j = 0; j < k; ++j) {
			f >> c;
			person[c]->father = person[t];
			person[c]->mother = person[m];
			person[t]->children->Add(person[c]);
		}
	}

	for(i = 1; i <= n; ++i)
		tree.Insert(person[i]);

	f >> q;
	for(k = 0; k < q; ++k) {
		f >> i >> j;
		tree.Relative(person[i]->name, person[j]->name);
	}

	f.close();
	g.close();
	return 0;
}

