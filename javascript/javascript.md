- Don't use obscure shortcuts like ```~~```
    
	```javascript
    // bad
    switch (~~this.getAttribute('data-group')) {
    
    // good
    switch (Math.floor(this.getAttribute('data-group'))) {
    ```
    
