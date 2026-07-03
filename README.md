# UAS_TeknikKompilasi
# Natan Nael
# Nim 231011400113

# Pembuatan Pattern (BNF)
<while_stmt>    ::= "while" "(" <condition> ")" "{" <stmt_list> "}"
<condition>     ::= <identifier> <relop> <value>
<stmt_list>     ::= <assignment> ";"
<assignment>    ::= <identifier> "=" <expression>
<expression>    ::= <identifier> <arith_op> <identifier>
<relop>         ::= "<" | ">" | "==" | "!="
<arith_op>      ::= "+" | "-" | "*" | "/"
<identifier>    ::= [a-zA-Z_][a-zA-Z0-9_]*
<value>         ::= [0-9]+

#  Implementasi Kode (Python)
import re

#==========================================
#1. ANALISIS LEKSIKAL (Lexer)
#==========================================
class Token:
    def __init__(self, type, value):
        self.type = type
        self.value = value
    def __repr__(self):
        return f"[{self.type}: {self.value}]"

def lexer(code):
    token_specification = [
        ('WHILE',    r'while'),
        ('LPAREN',   r'\('),
        ('RPAREN',   r'\)'),
        ('LBRACE',   r'\{'),
        ('RBRACE',   r'\}'),
        ('ID',       r'[a-zA-Z_][a-zA-Z0-9_]*'),
        ('NUMBER',   r'\d+'),
        ('ASSIGN',   r'='),
        ('OP',       r'[+\-*/]'),
        ('RELOP',    r'[<>!=]+'),
        ('SEMI',     r';'),
        ('SKIP',     r'[ \t\n]+'),
    ]
    tokens = []
    for type, pattern in token_specification:
        regex = re.compile(pattern)
    
    # Tokenizing using finditer
    pos = 0
    while pos < len(code):
        match = None
        for type, pattern in token_specification:
            regex = re.compile(pattern)
            match = regex.match(code, pos)
            if match:
                if type != 'SKIP':
                    tokens.append(Token(type, match.group()))
                pos = match.end()
                break
        if not match:
            raise SyntaxError(f"Karakter ilegal: {code[pos]}")
    return tokens

#==========================================
#2. ANALISIS SINTAKSIS (Parser -> AST)
#==========================================
class ASTNode: pass

class WhileNode(ASTNode):
    def __init__(self, condition, body):
        self.condition = condition
        self.body = body

class AssignmentNode(ASTNode):
    def __init__(self, target, expression):
        self.target = target
        self.expression = expression

class BinaryOpNode(ASTNode):
    def __init__(self, left, op, right):
        self.left = left
        self.op = op
        self.right = right

class Parser:
    def __init__(self, tokens):
        self.tokens = tokens
        self.pos = 0

    def consume(self, expected_type):
        if self.tokens[self.pos].type == expected_type:
            token = self.tokens[self.pos]
            self.pos += 1
            return token
        raise SyntaxError(f"Diharapkan {expected_type}, ditemukan {self.tokens[self.pos].type}")

    def parse(self):
        self.consume('WHILE')
        self.consume('LPAREN')
        cond_left = self.consume('ID').value
        cond_op = self.consume('RELOP').value
        cond_right = self.consume('NUMBER').value
        self.consume('RPAREN')
        
        self.consume('LBRACE')
        # Simple body parser (satu assignment)
        target = self.consume('ID').value
        self.consume('ASSIGN')
        expr_left = self.consume('ID').value
        expr_op = self.consume('OP').value
        expr_right = self.consume('ID').value
        self.consume('SEMI')
        self.consume('RBRACE')
        
        condition = BinaryOpNode(cond_left, cond_op, cond_right)
        body = AssignmentNode(target, BinaryOpNode(expr_left, expr_op, expr_right))
        return WhileNode(condition, body)

#==========================================
#3. ANALISIS SEMANTIK
#==========================================
def semantic_analyzer(node, symbol_table):
    # Cek apakah variabel dalam body sudah dideklarasikan di symbol_table
    # Untuk simulasi, kita cek variabel 'counter' dan 'sum'
    def check_id(name):
        if name not in symbol_table:
            raise NameError(f"Kesalahan Semantik: Variabel '{name}' belum dideklarasikan.")

    if isinstance(node, WhileNode):
        check_id(node.condition.left)
        if isinstance(node.body, AssignmentNode):
            check_id(node.body.target)
            check_id(node.body.expression.left)
            check_id(node.body.expression.right)
    print("Analisis Semantik: Berhasil (Semua variabel valid).")

#==========================================
#4. GENERASI KODE ANTARA (Three-Address Code)
#==========================================
def generate_tac(node):
    print("\n--- Three-Address Code (TAC) ---")
    label_start = "L1"
    label_end = "L2"
    
    cond = node.condition
    body = node.body
    
    print(f"{label_start}:")
    print(f"  if !({cond.left} {cond.op} {cond.right}) goto {label_end}")
    
    # Generate temp variable untuk ekspresi di dalam body
    temp = "t1"
    print(f"  {temp} = {body.expression.left} {body.expression.op} {body.expression.right}")
    print(f"  {body.target} = {temp}")
    
    print(f"  goto {label_start}")
    print(f"{label_end}:")
    print("--------------------------------")

#==========================================
#RUNNER
#==========================================
source_code = "while (counter < 10) { sum = sum + counter; }"

print(f"Source Code: {source_code}\n")

#1. Lexical
tokens = lexer(source_code)
print("1. Tokens:", tokens)

#2. Syntax
parser = Parser(tokens)
ast = parser.parse()
print("2. AST: WhileNode Berhasil Dibentuk.")

#3. Semantic
#Simulasi tabel simbol (variabel yang sudah ada di memori)
symbol_table = ['counter', 'sum']
semantic_analyzer(ast, symbol_table)

#4. Code Gen
generate_tac(ast)
Penjelasan Tahapan Kompilasi

A. Analisis Leksikal (Lexer)
Pada tahap ini, kode sumber string dipecah menjadi unit terkecil yang disebut Token. Program menggunakan Regular Expression untuk mengidentifikasi apakah sebuah kata termasuk kata kunci (while), operator (<, +), identifier (counter), atau simbol ({, }).
Hasil: Kumpulan token seperti [WHILE: while], [LPAREN: (], dst.


B. Analisis Sintaksis (Parser)
Tahap ini memeriksa apakah urutan token sesuai dengan aturan tata bahasa (Grammar) yang telah didefinisikan dalam BNF. Program menggunakan Recursive Descent Parsing sederhana untuk membangun Abstract Syntax Tree (AST).
Struktur AST: WhileNode memiliki anak berupa BinaryOpNode (untuk kondisi) dan AssignmentNode (untuk isi perulangan).


C. Analisis Semantik
Tahap ini memeriksa makna dari kode tersebut. Dalam implementasi ini, dilakukan pengecekan Variabel Scope/Declaration. Program memeriksa apakah variabel counter dan sum sudah ada dalam symbol_table. Jika kita menggunakan variabel yang tidak terdaftar, compiler akan memberikan error meskipun secara struktur (sintaksis) sudah benar.


D. Generasi Kode Antara (Three-Address Code)
Tahap akhir adalah mengubah AST menjadi format kode antara yang lebih dekat dengan bahasa mesin namun masih terbaca manusia. TAC menggunakan maksimal tiga alamat (dua operan dan satu hasil).
Logika TAC While:
Membuat label awal (L1).
Melakukan pengecekan kondisi. Jika salah, lompat ke akhir (goto L2).
Mengeksekusi isi loop (menggunakan variabel sementara t1).
Lompat kembali ke atas (goto L1) untuk pengecekan ulang.
Label akhir (L2).
