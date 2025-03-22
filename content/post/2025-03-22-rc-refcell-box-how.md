---
title: Use `Rc`, `RefCell`, `Box` in correct way
date: '2025-03-22'
categories:
    - post
tags:
    - rust 
draft: true
---

```rust
#[derive(Default, Clone, Debug)]
pub struct Environment {
    inner: HashMap<String, Expression>,
    parent: Option<Rc<RefCell<Environment>>>,
}

impl Environment {
    #[must_use]
    pub fn new(parent: Option<Environment>) -> Self {
        Environment {
            inner: HashMap::new(),
            parent: parent.map(|p| Rc::new(RefCell::new(p))),
        }
    }

    fn fork_from(parent: &Rc<RefCell<Self>>) -> Rc<RefCell<Environment>>  {
        Rc::new(RefCell::new(Environment {
            inner: HashMap::new(),
            parent: Some(Rc::clone(parent)),
        }))
    }

    fn insert(&mut self, k: String, v: Expression) -> Option<Expression> {
        self.inner.insert(k, v)
    }

    fn update(&mut self, k: String, v: Expression) -> Option<Expression> {
        match self.inner.get(&k) {
            Some(_) => self.insert(k, v),
            None => match self.parent {
                Some(ref parent) => parent.borrow_mut().update(k, v),
                _ => None,
            },
        }
    }

    fn get(&self, k: &String) -> Option<Expression> {
        match self.inner.get(k) {
            Some(expr) => Some(expr.clone()),
            None => match self.parent {
                Some(ref parent) => parent.borrow().get(k),
                _ => None,
            },
        }
    }
}

pub fn interpret(
    declarations: Vec<Declaration>,
    src: &str,
    env: &Rc<RefCell<Environment>>,
) -> Result<(), miette::Error> {
    for declaration in declarations {
        ...
        Stmt::Block(decls) => {
            let env = Environment::fork_from(&Rc::clone(env));
            interpret(decls, src, &env)?;
        }
        ...
    }
    Ok(())
}
```

