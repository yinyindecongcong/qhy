#include <iostream>
#include <fstream>
#include <sstream>
#include <set>
#include <vector>
#include <string>
#include <stack>
#include <queue>
#include <algorithm>
using namespace std;
//define the data member and operation of polynomial 
typedef vector<pair<int, int> > ITEMS;
class polynomial{
 public:
 	//construction and deconstruction
 	polynomial();
 	polynomial(const string & s1, const string & s2 = "");
 	polynomial(const polynomial& P);
 	polynomial& operator = (const polynomial& P);
 	~polynomial();
 	//sort by name
 	bool operator < (const polynomial& B) const;
 	//add
 	friend polynomial operator + (const polynomial & A, const polynomial & B);
 	friend polynomial operator - (const polynomial & A, const polynomial & B);
 	friend polynomial operator * (const polynomial & A, const polynomial & B);
 	friend bool operator == (const polynomial & A, const polynomial & B);
 	polynomial getDerivative() const;
 	int get_value(int x)const;
 	//be used to write to a file and store
 	ostream& writeFile(ostream & out)const;
 	//input and out put a polynomial

 	//determine if it is legal input and legal item of a polynomial
 	bool judge(string & s);
 	int read();
 	void print() const;
 public:
 	string name;
 private:
 	bool isLegalitem(int a, int b);
 	polynomial mulItem(const pair <int, int> &p)const ;
 	ITEMS items;
};

class SetOfPoly{
 public:
 	SetOfPoly(){}
 	int numOfPoly();
 	//initialize the Set through the file which stores polynomials
 	bool initSet();
 	//save the Set of polynomials into a file
 	bool saveSet();
 	//add a polynomial
 	void insertPolynomial(const polynomial & P);
 	//remove a polynomial by name
 	bool erasePolynomial(const string & s);
 	//find a polynomial by name and return the point if exists
 	bool findPolynomial(const string & s, polynomial *& Pit);
 	//print a polynomial by name
 	bool printPolynomial(const string & s);
 	//print all polynomial in the Set
 	void printALL();
 	//determine if it is print statement
 	bool judgePrint(const string & s, string & p);
 	~SetOfPoly();
 private:
 	set <polynomial> Set;
};
class calculator{
 public:
 	calculator(){ S.initSet(); }
 	//start the calculator
 	void start();
 	void MenuPrint();
 	void ifsave();
 private:
 	//choose the operation
 	char MemuSelect();
 	//operations
 	int start_1();
 	//pre:input a polynomial with name
 	//pos:store it , or output error message if is illegal

 	int start_2();
 	//pre:input the name of polynomial
 	//pos:show the polynomial or output error message if is not exist

 	bool AUstart(const string & s, string & p, string & q, string & r, char op);
 	//auxiliary function of start_3/4/5, determine if the formula is legal
 	
 	int start_3();
 	//pre:input a formula of addition
 	//pos:show the result and, if the name of result is given, store it

 	int start_4();
 	//pre:input a formula of subtraction
 	//pos:show the result and, if the name of result is given, store it

	int start_5();
 	//pre:input a formula of multiplication
 	//pos:show the result and, if the name of result is given, store it

 	bool AUstart_6(const string & s, string & p, string & q);
 	//auxiliary function of start_6, determine if the formula is legal

 	int start_6();
 	//pre:input a judgement of equalition
 	//pos:determine if two polynomial is equal

 	bool AUstart_7(const string & s, string & p, string & r);
 	//auxiliary function of start_7, determine if the formula is legal

 	int start_7();
 	//get derivative

 	bool AUstart_8(const string & s, string & p, int & x);
 	//auxiliary function of start_8, determine if the formula is legal

 	int start_8();
 	//get value of given x

 	void start_x();
 	bool poly_operations(const string & s, queue<string> & Q);

 	bool isLegalName(const string & s);

  private:
 	SetOfPoly S;
};

int main(){
	calculator C;
	C.start();
	C.ifsave();
	return 0;
}
polynomial::polynomial(){}
polynomial::polynomial(const string & s1, const string & s2):name(s1){
	stringstream sin(s2);
	int a, b;
	while(sin >> a && sin >> b){
		pair<int, int> p = make_pair(a, b);
		items.push_back(p);
	}
}
polynomial::polynomial(const polynomial& P):name(P.name), items(P.items.begin(), P.items.end()){}
polynomial& polynomial::operator = (const polynomial& P){
	name = P.name;
	items.clear();
	for (ITEMS::const_iterator it = P.items.begin(); it != P.items.end(); it++)
		items.push_back(*it);
	return *this;
}
bool polynomial::operator < (const polynomial& B) const{
	return name < B.name;
}
polynomial operator + (const polynomial & A, const polynomial & B){
	polynomial sum;
	unsigned int i = 0, j = 0;
	while(i < A.items.size() && j < B.items.size()){
		if (A.items[i].second > B.items[j].second){
			sum.items.push_back(A.items[i++]);
		}
		else if (A.items[i].second < B.items[j].second){
			sum.items.push_back(B.items[j++]);
		}
		else {
			int a = A.items[i].first + B.items[j].first;
			if (a) {
				pair<int, int> p = make_pair(a, A.items[i].second);
				sum.items.push_back(p);
			}
			i++; 
			j++;
		}
	}
	while(i < A.items.size()) sum.items.push_back(A.items[i++]);
	while(j < B.items.size()) sum.items.push_back(B.items[j++]);
	return sum;
}
polynomial operator - (const polynomial & A, const polynomial & B){
	polynomial sum;
	unsigned int i = 0, j = 0;
	while(i < A.items.size() && j < B.items.size()){
		if (A.items[i].second > B.items[j].second){
			sum.items.push_back(A.items[i++]);
		}
		else if (A.items[i].second < B.items[j].second){
			pair<int, int> p = make_pair(-B.items[j].first, B.items[j].second);
			sum.items.push_back(p);
			j++;
		}
		else {
			int a = A.items[i].first - B.items[j].first;
			if (a) {
				pair<int, int> p = make_pair(a, A.items[i].second);
				sum.items.push_back(p);
			}
			i++; 
			j++;
		}
	}
	while(i < A.items.size()) sum.items.push_back(A.items[i++]);
	while(j < B.items.size()) {
		pair<int, int> p = make_pair(-B.items[j].first, B.items[j].second);
		sum.items.push_back(p);
		j++;
	}
	return sum;
}
bool operator == (const polynomial & A, const polynomial & B){
	if (A.items.size() != B.items.size()) return 0;
	for (unsigned int i = 0; i < A.items.size(); i++)
		if (A.items[i] != B.items[i]) return 0;
	return 1;
}
polynomial operator * (const polynomial & A, const polynomial & B){
	polynomial result;
	for (unsigned int i = 0; i < B.items.size(); i++)
		result = result + A.mulItem(B.items[i]);
	return result;
}
polynomial polynomial::mulItem(const pair<int, int> & p)const{
	polynomial result;
	for (unsigned int i = 0; i < items.size(); i++){
		pair <int, int> n = make_pair(items[i].first * p.first, items[i].second + p.second);
		result.items.push_back(n);
	}
	return result;
}
int polynomial::get_value(int x)const{
	int result = 0;
	for (unsigned int i = 0; i < items.size(); i++){
		double xsquare = pow(x, items[i].second);
		result += items[i].first * xsquare;
	}
	return result;
}
polynomial polynomial::getDerivative() const{
	polynomial R;
	for (unsigned int i = 0; i < items.size(); i++){
		if (items[i].second > 0) {
			pair<int, int> p = make_pair(items[i].second * items[i].first, items[i].second - 1);
			R.items.push_back(p);
		}
	}
	return R;
}
bool isNum(string s, int & result){
	if (s == "0") { result = 0; return 1; }
	bool sign = 0;
	if (s[0] == '-') sign = 1;
	if (s[sign] < '1' || s[sign] > '9') return 0;
	else result = s[sign] - '0';
	unsigned int i = sign + 1;
	while(i < s.size()){
		if (s[i] < '0' || s[i] > '9') return 0;
		else result = result * 10 + s[i] - '0';
		i++;
	}
	if (sign) result = -result;
	return 1;
}
bool polynomial::isLegalitem(int a, int b){
	if (a == 0 || b < 0) return 0;
	if (!items.empty() && b >= items[items.size() - 1].second) return 0;
	return 1;
}
bool polynomial::judge(string & s){
	stack<char> match;
	string str = "";
	int a, b;
	bool eqSign = 0;
	for (int i = 0; i < (int)s.size(); i++)
		if (s[i] == '\t' || s[i] == ' ') continue;
		else if (s[i] == '=' && !eqSign) { name = str; eqSign = 1; str = ""; }
		else if (s[i] == '('){
			if (!match.empty()) return 0;
			match.push('(');
		}
		else if (s[i] == ','){
			if (match.top() != '(' || str.empty() || !isNum(str, a)) return 0;
			str = "";
			match.pop();
			match.push(',');
		}
		else if (s[i] == ')'){
			if (match.top() != ',' || str.empty() || !isNum(str, b)) return 0;
			if (!isLegalitem(a, b)) return 0;
			pair<int, int> pi = make_pair(a, b);
			items.push_back(pi);
			str = "";
			match.pop();
		}
		else str += s[i];
	if (items.empty() || str != "") return 0;
	return 1;
}
int polynomial::read(){
	/*
	input the polynomial in the format like (c3,e3)(c2,e2)(c1,e1), which cs represent coefficients and es represent exponents.
	and exponents must be in descending order and not less than 0
	*/
	items.clear();
	string s;
	bool flag = 0;
	do {
		getline(cin, s, '\n');
		if (s == "0") return 2;
		if (!judge(s)) {
			cout << "illegal input, please try again" << endl;
			items.clear();
			s = "";
			name = "";
			flag = 1;
		}
		else break;
	}while(flag);
	return 1;
}
void polynomial::print() const{
	if (name != "") cout << name << " = ";
	if (items.empty()) { cout << '0' << endl; return; }
	bool first = 1;
	ITEMS::const_iterator it = items.begin();
	while(it != items.end()){
		if ((*it).first > 0){
			if (!first) cout << " + ";
			if ((*it).second == 0) cout << (*it).first;
			else{
				if ((*it).first != 1) cout << (*it).first;
				cout << "x";
				if ((*it).second != 1) cout << '^' << (*it).second;
			} 
		}
		else if ((*it).first < 0){
			cout << " - ";
			if ((*it).second == 0) cout << -(*it).first;
			else{
				if ((*it).first != -1) cout << -(*it).first;
				cout << "x";
				if ((*it).second != 1) cout << '^' << (*it).second;
			} 
		}
		first = 0;
		it++;
	}
	cout << endl;
}
ostream& polynomial::writeFile(ostream & out)const {
	out << name << ' ';
	for (unsigned int i = 0; i < items.size(); i++)
		out << items[i].first << ' ' << items[i].second << ' ';
	out << endl;
	return out;
}
polynomial::~polynomial(){
	items.clear();
}
int SetOfPoly::numOfPoly(){ return Set.size(); }
bool SetOfPoly::initSet(){
	ifstream in("x.txt");
	if (!in) return 0;
	string s1, s2;
	while (in >> s1 && getline(in, s2)){
		polynomial P(s1, s2);
		Set.insert(P);
	}
	in.close();
	return 1;
}
bool SetOfPoly::saveSet(){
	ofstream out("x.txt");
	if (!out) return 0;
	for (set <polynomial>::iterator it = Set.begin(); it != Set.end(); it++) it->writeFile(out);
	out.close();
	return 1;
}
void SetOfPoly::insertPolynomial(const polynomial & P){ Set.insert(P); }
bool SetOfPoly::erasePolynomial(const string & s){
	polynomial P(s);
	set <polynomial>::iterator it = Set.find(P);
	if(it == Set.end()) return 0;
	Set.erase(*it);
	return 1;
}
bool SetOfPoly::findPolynomial(const string & s, polynomial*& Pit){
	polynomial P(s);
	set <polynomial>::iterator it = Set.find(P);
	if(it == Set.end()) return 0;
	Pit = const_cast<polynomial*>(&(*it));
	return 1;
}
bool SetOfPoly::printPolynomial(const string & s){
	polynomial P(s);
	set <polynomial>::iterator it = Set.find(P);
	if(it == Set.end()) return 0;
	cout << "the polynomial is : ";
	(*it).print();
	return 1;
}
void SetOfPoly::printALL(){
	cout << "ALL polynomial(s) :" << endl;
	for (set <polynomial>::iterator it = Set.begin(); it != Set.end(); it++)
		(*it).print();
}
bool SetOfPoly::judgePrint(const string & s, string & p){
	if (s.size() < 7 || s.substr(0,5) != "print" || s[5] != ' ') return 0;
	string si = s.substr(6, s.size() - 6);
	stringstream sin(si);
	getline(sin, p);
	return 1; 
}
SetOfPoly::~SetOfPoly(){}
char calculator::MemuSelect(){
	char c;
	cin >> c;
	cin.clear();
	cin.sync();
	return c;
}
void calculator::MenuPrint(){
	cout << "POLYNOMIAL-CALCULATOR" << endl;
	cout << "--------------------------------------" << endl;
	cout << "1.input a polynomial" << endl;
	cout << "2.show a polynomial" << endl;
	cout << "3.addition of two polynomials" << endl;
	cout << "4.subtraction of two polynomials" << endl;
	cout << "5.multiplication of two polynomials" << endl;
	cout << "6.equal-determination" << endl;
	cout << "7.get derivative" << endl;
	cout << "8.get value" << endl;
	cout << "9.exit" << endl;
	cout << "0.show menu" << endl;
	cout << "x.command mode" << endl;
	cout << "choose your operation above" << endl;
	cout << "--------------------------------------" << endl;
}
int calculator::start_1(){
	system("cls");
	cout << "POLYNOMIAL-CALCULATOR->INPUT" << endl;
	cout << "--------------------------------------" << endl;
	cout << "input the polynomial in the format like 'p = (c3,e3)(c2,e2)(c1,e1)', " << endl;
	cout << "which c represent coefficients and e represent exponents." << endl;
	cout << "and exponents must be in descending order and not less than 0" << endl;
	cout << "(input 0 to return to menu)" << endl;
	cout << "--------------------------------------" << endl;
	polynomial P;
	if (P.read() == 2) return 2;
	if (P.name == ""){
		cout << "error--there is no a name for the polynomial" << endl;
		cout << "please input again" << endl;
		start_1();
	}
	else {
		cout << "the polynomial is : ";
		P.print();
		S.insertPolynomial(P);
	}	
	return 1;
}
int calculator::start_2(){
	system("cls");
	cout << "POLYNOMIAL-CALCULATOR->PRINT" << endl;
	cout << "--------------------------------------" << endl;
	cout << "input the name of the polynomial to show" << endl;
	cout << "or input 'all' to show all polynomials" << endl;
	cout << "(input 0 to return to menu)" << endl;
	cout << "--------------------------------------" << endl;
	string s;
	getline(cin, s);
	if (s == "0") return 2;
	if (s == "all") S.printALL();
	else if (!S.printPolynomial(s)) cout << "error--there is no polynomial named " << s << endl;
	return 1;
}
bool calculator::isLegalName(const string & s){
	if (!((s[0] >= 'a' && s[0] <= 'z')||(s[0] >= 'A' && s[0] <= 'Z'))) return 0;
	for (unsigned int i = 1; i < s.size(); i++)
		if (!((s[i] >= 'a' && s[i] <= 'z') || (s[i] >= 'A' && s[i] <= 'Z') || (s[i] >= '0' && s[i] <= '9'))) return 0;
	return 1;
}
bool calculator::AUstart(const string & s, string & p, string & q, string & r, char op){
	string str = "";
	bool eqSign = 0;
	unsigned int i = 0;
	for (; i < s.size(); i++){
		if (s[i] == ' ' || s[i] == '\t') continue;
		else if (s[i] == '='){
			if (eqSign || str == "" || !isLegalName(str)) return 0;
			eqSign = 1;
			r = str;
			str = "";
		}
		else if (s[i] == op){
			if (str == "" || !isLegalName(str)) return 0;
			p = str;
			str = "";
			break;
		}
		else str += s[i];
	}
	for (i = i + 1; i < s.size(); i++)
		if (s[i] != ' ' && s[i] != '\t') str += s[i];
	if (isLegalName(str)) q = str;
	if (p == "" || q == "") return 0;
	return 1;
}
int calculator::start_3(){
	system("cls");
	cout << "POLYNOMIAL-CALCULATOR->ADDITION" << endl;
	cout << "--------------------------------------" << endl;
	cout << "input in the format like 'p + q' or 'r = p + q'" << endl;
	cout << "which p, q, r are the name of the polynomial" << endl;
	cout << "the latter way will store the result with name r" << endl;
	cout << "(input 0 to return to menu)" << endl;
	cout << "--------------------------------------" << endl;
	string s, r = "", p = "", q = "";
	getline(cin, s);
	if (s == "0") return 2;
	if (AUstart(s, p, q, r, '+')){
		polynomial * Pit = NULL, * Qit = NULL;
		if (S.findPolynomial(p, Pit) && S.findPolynomial(q, Qit)){
			polynomial R = (*Pit) + (*Qit);
			if (r != "") { R.name = r; S.insertPolynomial(R); }
			cout << "The result is : ";  R.print();
			return 1;
		}
		else cout << "error--there is no polynomial named " << p << "/" << q << endl;
	}
	else {
		cout << "error--illegal input" << endl;
	}
	return 0;
}
int calculator::start_4(){
	system("cls");
	cout << "POLYNOMIAL-CALCULATOR->SUBTRACTION" << endl;
	cout << "--------------------------------------" << endl;
	cout << "input in the format like 'p - q' or 'r = p - q'" << endl;
	cout << "which p, q, r are the name of the polynomial" << endl;
	cout << "the latter way will store the result with name r" << endl;
	cout << "(input 0 to return to menu)" << endl;
	cout << "--------------------------------------" << endl;
	string s, r = "", p = "", q = "";
	getline(cin, s);
	if (s == "0") return 2;
	if (AUstart(s, p, q, r, '-')){
		polynomial * Pit = NULL, * Qit = NULL;
		if (S.findPolynomial(p, Pit) && S.findPolynomial(q, Qit)){
			polynomial R = (*Pit) - (*Qit);
			if (r != "") { R.name = r; S.insertPolynomial(R); }
			cout << "The result is : ";  R.print();
			return 1;
		}
		else cout << "error--there is no polynomial named " << p << "/" << q << endl;
	}
	else {
		cout << "error--illegal input" << endl;
	}
	return 0;
}
int calculator::start_5(){
	system("cls");
	cout << "POLYNOMIAL-CALCULATOR->MULTIPLICATION" << endl;
	cout << "--------------------------------------" << endl;
	cout << "input in the format like 'p * q' or 'r = p * q'" << endl;
	cout << "which p, q, r are the name of the polynomial" << endl;
	cout << "the latter way will store the result with name r" << endl;
	cout << "(input 0 to return to menu)" << endl;
	cout << "--------------------------------------" << endl;
	string s, r = "", p = "", q = "";
	getline(cin, s);
	if (s == "0") return 2;
	if (AUstart(s, p, q, r, '*')){
		polynomial * Pit = NULL, * Qit = NULL;
		if (S.findPolynomial(p, Pit) && S.findPolynomial(q, Qit)){
			polynomial R = (*Pit) * (*Qit);
			if (r != "") { R.name = r; S.insertPolynomial(R); }
			cout << "The result is : "; R.print();
			return 1;
		}
		else cout << "error--there is no polynomial named " << p << "/" << q << endl;
	}
	else {
		cout << "error--illegal input" << endl;
	}
	return 0;
}
bool calculator::AUstart_6(const string & s, string & p, string & q){
	string str = "";
	unsigned int i = 0;
	for (; i < s.size(); i++)
		if (s[i] == ' ' || s[i] == '\t') continue;
		else if (s[i] == '='){
			if (i + 1 == s.size() || s[i + 1] != '=' || str == "" || !isLegalName(str)) return 0;
			p = str;
			str = "";
			break;
		}
		else str += s[i];
	if (i + 2 < s.size()) 
		for (i = i + 2; i < s.size(); i++)
			if (s[i] == ' ' || s[i] == '\t') continue;
			else q += s[i];
	if (p == "" || q == "") return 0;
	return 1;
}
int calculator::start_6(){
	system("cls");
	cout << "POLYNOMIAL-CALCULATOR->COMPARISON" << endl;
	cout << "--------------------------------------" << endl;
	cout << "input the two compared polynomials in the format like 'p == q'" << endl;
	cout << "(input 0 to return to menu)" << endl;
	cout << "--------------------------------------" << endl;
	string s, p = "", q = "";
	polynomial * Pit = NULL, *Qit = NULL;
	getline(cin, s);
	if (s == "0") return 2;
	//determine if it is legal
	if (AUstart_6(s, p, q)){
		if (S.findPolynomial(p, Pit) && S.findPolynomial(q, Qit)){
			if (*Pit == *Qit) cout << "The two polynomials are euqal." << endl;
			else cout << "The two polynomials are NOT euqal." << endl;
			return 1;
		}
		else cout << "error--there is no polynomial named " << p << "/" << q << endl;
	}
	else {
		cout << "error--illegal input" << endl;
	}
	return 0;
}
bool calculator::AUstart_7(const string & s, string & p, string & r){
	string str = "";
	unsigned int i = 0;
	for (; i < s.size(); i++)
		if (s[i] == ' ' || s[i] == '\t') continue;
		else if (s[i] == '='){
			if (str == "" || !isLegalName(str)) return 0;
			r = str;
			str = "";
			break;
		}
		else str += s[i];
	for (i = i + 1; i < s.size(); i++)
		if (s[i] == ' ' || s[i] == '\t') continue;
		else str += s[i];
	if (str.size() < 2 || str[str.size() - 1] != '\'') return 0;
	p = str.substr(0, str.size() - 1);
	return 1;
}
int calculator::start_7(){
	system("cls");
	cout << "POLYNOMIAL-CALCULATOR->DERIVATIVE" << endl;
	cout << "--------------------------------------" << endl;
	cout << "input in the format like ' p' ' or ' r = p' '" << endl;
	cout << "which p, r are the name of the polynomial" << endl;
	cout << "the latter way will store the result with name r" << endl;
	cout << "(input 0 to return to menu)" << endl;
	cout << "--------------------------------------" << endl;
	string s = "", p = "", r = "";
	polynomial * Pit = NULL;
	getline(cin, s);
	if (s == "0") return 2;
	//determine if it is legal
	if (AUstart_7(s, p, r)){
		if (S.findPolynomial(p, Pit)){
			polynomial R = Pit->getDerivative();
			if (r != "") { R.name = r; S.insertPolynomial(R); }
			cout << "the derivative of " << p << " is : ";  R.print();
			return 1;
		}
		else cout << "error--there is no polynomial named " << p << endl;
	}
	else {
		cout << "error--illegal input" << endl;
	}
	return 0;
}
bool calculator::AUstart_8(const string & s, string & p, int & x){
	string str = "";
	unsigned int i = 0;
	bool left = 0, right = 0;
	for (; i < s.size(); i++)
		if (s[i] == ' ' || s[i] == '\t') continue;
		else if (s[i] == '['){
			if (left++ || str == "" || !isLegalName(str)) return 0;
			p = str;
			str = "";
		}
		else if (s[i] == ']'){
			if (left != 1 || right++ || str.size() <= 2 || str[0] != 'x' || str[1] != '=' || !isNum(str.substr(2,str.size() - 2), x)) return 0;
			str = "";
		}
		else str += s[i];
	if (str != "" || !left || !right) return 0;
	return 1;
}
int calculator::start_8(){
	system("cls");
	cout << "POLYNOMIAL-CALCULATOR->GET_VALUE" << endl;
	cout << "--------------------------------------" << endl;
	cout << "input in the format like 'p[x=3]'" << endl;
	cout << "which p is the name of the polynomial" << endl;
	cout << "(input 0 to return to menu)" << endl;
	cout << "--------------------------------------" << endl;
	string s = "", p = "";
	polynomial * Pit = NULL;
	int x = 0;
	getline(cin, s);
	if (s == "0") return 2;
	//determine if it is legal
	if (AUstart_8(s, p, x)){
		if (S.findPolynomial(p, Pit)){
			cout << "the result is :" << Pit->get_value(x) << endl;
			return 1;
		}
		else cout << "error--there is no polynomial named " << p << endl;
	}
	else {
		cout << "error--illegal input" << endl;
	}
	return 0;
}
void calculator::start(){
	system("cls");
	MenuPrint();
	char c;
	int i;
	switch(MemuSelect()){
		case '1':
			i = start_1();
			if (i == 2) goto case0;
			cout << "enter any key to show menu" << endl;
			cin.get(c);
			start();
			break;
		case '2':
			i = start_2();
			if (i == 2) goto case0;
			cout << "enter any key to show menu" << endl;
			cin.get(c);
			start();
			break;
		case1: case '3':
			i = start_3();
			if (i == 2) goto case0;
			if (!i) cout << "enter 'Y/y' to input again" << endl;
			else cout << "enter any key to show menu" << endl;
			c = MemuSelect();
			if (c == 'Y' || c == 'y') goto case1;
			start();
			break;
		case2: case '4':
			i = start_4();
			if (i == 2) goto case0;
			if (!i) cout << "enter 'Y/y' to input again" << endl;
			else cout << "enter any key to show menu" << endl;
			c = MemuSelect();
			if (c == 'Y' || c == 'y') goto case2;
			start();
			break;
		case3: case '5':
			i = start_5();
			if (i == 2) goto case0;
			else if (!i) cout << "enter 'Y/y' to input again" << endl;
			else cout << "enter any key to show menu" << endl;
			c = MemuSelect();
			if (c == 'Y' || c == 'y') goto case3;
			start();
			break;
		case4: case '6':
			i = start_6();
			if (i == 2) goto case0;
			if (!i) cout << "enter 'Y/y' to input again" << endl;
			else cout << "enter any key to show menu" << endl;
			c = MemuSelect();
			if (c == 'Y' || c == 'y') goto case4;
			start();
			break;
		case5: case '7':
			i = start_7();
			if (i == 2) goto case0;
			else if (!i) cout << "enter 'Y/y' to input again" << endl;
			else cout << "enter any key to show menu" << endl;
			c = MemuSelect();
			if (c == 'Y' || c == 'y') goto case5;
			start();
			break;
		case6: case '8':
			i = start_8();
			if (i == 2) goto case0;
			else if (!i) cout << "enter 'Y/y' to input again" << endl;
			else cout << "enter any key to show menu" << endl;
			c = MemuSelect();
			if (c == 'Y' || c == 'y') goto case6;
			start();
			break;
		case '9':
			return;
		case '0':
			case0:
			start();
			break;
		case 'x':
			start_x();
			start();
			break;
		default:
			cout << "error--illegal input" << endl;
			cout << "enter any key to show menu" << endl;
			cin.get(c);
			start();
	}
}
//a - (b - c) * d
bool calculator::poly_operations(const string & s, queue<string> & Q){
	string str = "";
	stack<char> ST;
	bool right = 0;//if last one is right bracket
	for (unsigned int i = 0; i < s.size(); i++){
		if (s[i] == ' ' || s[i] == '\t') continue;
		else if (s[i] == '('){
			if (str  != "") return 0;
			ST.push(s[i]);
			right = 0;
		}
		else if (s[i] == ')'){
			if (str == "" || !isLegalName(str)) return 0;
			Q.push(str);
			str = "";
			while (!ST.empty() && ST.top() != '('){
				str += ST.top();
				Q.push(str);
				str = "";
				ST.pop();
			}
			if (ST.empty()) return 0;
			ST.pop();
			right = 1;
		}
		else if (s[i] == '+' || s[i] == '-'){
			if (right && str != "") return 0;
			if (!right && (str == ""|| !isLegalName(str))) return 0; 
			if (str != "") { Q.push(str); str = ""; }
			while(!ST.empty() && (ST.top() == '*' || ST.top() == '+' || ST.top() == '-')){
				str += ST.top();
				Q.push(str);
				str = "";
				ST.pop();
			}
			ST.push(s[i]);
			right = 0;
		}
		else if (s[i] == '*'){
			if (right && str != "") return 0;
			if (!right && (str == ""|| !isLegalName(str))) return 0; 
			if (str != "") { Q.push(str); str = ""; }
			while(!ST.empty() && ST.top() == '*'){
				str += ST.top();
				Q.push(str);
				str = "";
				ST.pop();
			}
			ST.push(s[i]);
			right = 0;
		}
		else { str += s[i]; right = 0; } 
	}
	if (right && str != "") return 0;
	if (!right && (str == "" || !isLegalName(str))) return 0;
	if (str != "") { Q.push(str); str = ""; }
	while(!ST.empty()){
		str += ST.top();
		Q.push(str);
		str = "";
		ST.pop();
	}
	if (Q.size() <= 2) return 0;
	return 1;
}
void calculator::start_x(){
	static int flag = 0;
	if (flag == 0){
		system("cls");
		cout << "POLYNOMIAL-CALCULATOR->COMMAND MODE" << endl;
		cout << "--------------------------------------" << endl;
		cout << "this is the command mode of the calculator" << endl;
		cout << "you can get instruction by input 'help' " << endl;
		cout << "(input '0' to return to the menu)" << endl;
		cout << "--------------------------------------" << endl;
		flag = 1;
	}
	string s = "", p = "", q = "", r = "";
	int x;
	getline(cin, s);
	polynomial R;
	queue <string> Q;
	//calculate polynomials
	if (poly_operations(s, Q)){
		stack <string> ST;
		stack <polynomial> SP;
		while (!Q.empty()){
			if (Q.front() == "+" || Q.front() == "-" || Q.front() == "*"){
				polynomial B = SP.top(); SP.pop();
				polynomial A = SP.top(); SP.pop();
				polynomial R;
				if (Q.front() == "+") R = A + B;
				else if (Q.front() == "-") R = A - B;
				else if (Q.front() == "*") R = A * B;
				SP.push(R);
			}
			else {
				polynomial * Pit = NULL;
				if (S.findPolynomial(Q.front(), Pit)) SP.push(*Pit);
				else { cout << "error--there is no polynomial named " << Q.front() << endl; break; }
			}
			Q.pop();
		}
		if (Q.empty() && SP.size() == 1) {
			cout << "the result is : ";
			SP.top().print();  
		}
	}
	//judge equaliation
	else if (AUstart_6(s, p, q)){
		polynomial * Pit = NULL, * Qit = NULL;
		if (S.findPolynomial(p, Pit) && S.findPolynomial(q, Qit)){
			if (*Pit == *Qit) cout << "The two polynomials are euqal." << endl;
			else cout << "The two polynomials are NOT euqal." << endl;
		}
		else cout << "error--there is no polynomial named " << p << "/" << q << endl;
	}
	//get derivative
	else if (AUstart_7(s, p, r)){
		polynomial * Pit = NULL;
		if (S.findPolynomial(p, Pit)){
			polynomial R = Pit->getDerivative();
			if (r != "") { R.name = r; S.insertPolynomial(R); }
			cout << "the derivative of " << p << " is : "; R.print();
		}
		else cout << "error--there is no polynomial named " << p << endl;
	}
	//get value
	else if (AUstart_8(s, p, x)){
		polynomial * Pit = NULL;
		if (S.findPolynomial(p, Pit)){
			cout << "the result is : " << Pit->get_value(x) << endl;
		}
		else cout << "error--there is no polynomial named " << p << endl;
	}
	//input a polynomial
	else if (R.judge(s)){
		if (R.name == ""){
		cout << "error--there is no a name for the polynomial" << endl;
		cout << "please input again" << endl;
	}
		else {
			cout << "the polynomial is : ";
			R.print();
			S.insertPolynomial(R);
		}	
	}
	//print polynomial(s)
	else if (S.judgePrint(s, p)){
		polynomial * Pit = NULL;
		if (p == "all") S.printALL();
		else if (S.findPolynomial(p, Pit)) Pit->print();
		else cout << "error--there is no polynomial named " << p << endl; 
	}
	else if (s == "help"){
		cout << "this is the command mode of the calculator, where you can: " << endl;
		cout << "A.input the polynomial in the format like 'p = (c3,e3)(c2,e2)(c1,e1)'" << endl;
		cout << "B.print the polynomial in the format like 'print p' or 'print all'" << endl;
		cout << "C.calculate polynomial(s) in the format like 'p + q', 'p - q', 'p * q' or 'p * (q + r) - s'" << endl;
		cout << "D.get derivative and get value in the format like ' p' ', ' r = p' ' or 'p[x=3]'" << endl;
		cout << "E.get instruction by input 'help' " << endl;
		cout << "F.exit command mode by input '0'" << endl;
	}
	else if (s == "0") {
		cout << "command mode exit" << endl;
		flag = 0;
		return;
	}
	else if (s == "") {}
	else cout << "error--illegal input" << endl << "please input again" << endl;
	start_x();
}
void calculator::ifsave(){
	cout << "input Y/N to choose if save these polynomials in file " << endl;
	char c = MemuSelect();
	if (c == 'Y' || c == 'y') { S.saveSet(); cout << "save successful." << endl; }
	else cout << "exit without save" << endl;
}
