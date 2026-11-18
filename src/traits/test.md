# Test
```
impl<T> TreeNode<T> {
    fn traverse_inorder<B>(&self, mut f: impl FnMut(&T) -> ControlFlow<B>) -> ControlFlow<B> {
        if let Some(left) = &self.left {
            left.traverse_inorder(&mut f)?;
        }
        f(&self.value)?;
        if let Some(right) = &self.right {
            right.traverse_inorder(&mut f)?;
        }
        ControlFlow::Continue(())
    }
}
```
