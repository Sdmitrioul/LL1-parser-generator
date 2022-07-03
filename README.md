# LL(1) parser generator
Top-down Kotlin parser generator for LL(1) grammars with support for synthesized and inherited attributes.

Grammar example:
```
run

@IMP [
    { util.fac }
]

run returns [v : Int]
            = addSub {$v = addSub.v}
              END
            ;

addSub returns [v : Int]
            = mulDiv addSub_({mulDiv.v})
              {$v = addSub_.v}
            ;

addSub_[left : Int] returns [v : Int]
            = ADD mulDiv    {val next = left!! + mulDiv.v!!}
              addSub_(next) {$v = addSub_.v}
            | SUB mulDiv    {val next = left!! - mulDiv.v!!}
              addSub_(next) {$v = addSub_.v}
            | {$v = left}
            ;

mulDiv returns [v : Int]
            = log mulDiv_({log.v}) {$v = mulDiv_.v}
            ;

mulDiv_[left : Int] returns [v : Int]
            = MUL log mulDiv_({left!! * log.v!!}) {$v = mulDiv_.v}
            | DIV log mulDiv_({left!! / log.v!!}) {$v = mulDiv_.v}
            | {$v = left}
            ;

log returns [v : Int]
            = unary log_({unary.v}) {$v = log_.v}
            ;

log_[left : Int] returns [v : Int]
            = LOG log {$v = (Math.log(left!!.toDouble()) / Math.log(log.v!!.toDouble())).toInt()}
            | {$v = left}
            ;

unary returns [v : Int]
            = SUB unary {$v = -unary.v!!}
            | first_pr {$v = first_pr.v}
            ;

first_pr returns [v: Int]
            = first {val f = first!!.v}
              fact({f}) { $v = fact!!.v }
            ;

first  returns [v: Int]
            = OP addSub CP {$v = addSub.v}
            | NUM          {$v = Integer.valueOf(NUM)}
            ;

fact[p: Int] returns [v: Int]
            = FAC { val pr = fac(p!!) }
              fact({pr}) { $v = fact.v }
            | { $v = p }
            ;

ADD = "+";
SUB = "-";
MUL = "*";
LOG = "log";
DIV = "/";
OP = "(";
CP = ")";
NUM : "([1-9][0-9]*)|(0)";
FAC = "!";
END = "$";
```

- Blocks of code (in Kotlin) are writing in {}
- Terminals with constant value set by `<TERM-NAME> = "<String>"`
- Terminals with regexp set by `<TERM-NAME> : "<RegExp>"`
- The type of the synthesized attribute is written after the name of the rule as `returns [<NameOfPar> : <Type>]`
- The header (for imports) is defined via `@IMP [
  <code fragments>
  ]`
- Starting NonTerminal set in first line.
- Output rules for non-terminals are written with `|`
