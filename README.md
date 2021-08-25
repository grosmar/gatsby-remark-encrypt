# gatsby-remark-encrypt
Encrypt your markdown posts in Gatsby website.

## Install
``` bash
npm install --save gatsby-remark-encrypt crypto-js
```

## How to use
Add plugin at the **LAST** of plugins array in *gatsby-transform-remark*. 

Because the plugin needs to encrypt the whole html after other plugins' process finish.

``` diff
// In your gatsby-config.js
plugins: [
  {
    resolve: `gatsby-transformer-remark`,
    options: {
      plugins: [
        ...otherPlugins,
+       {
+          resolve: `gatsby-remark-encrypt`,
+          options: {
+            password: "123456"
+          }
+        }
+      ],
    },
  },
]
```

Add `password` field in frontmatter of the post you want to encrypt.
If you want to use individual password, set any password, or set it to TRUE in case you want to use the plugin global password

``` diff
  ---
  title: My encrypted post
  date: 2020-12-05 21:00:00
+ password: 123456
  ---
```

In your page query, add the `encrypted` field to tell whether the post is encrypted.

NOTE: Do not query `password` in frontmatter, or the raw password may leaked in page-data.json.

``` diff
// In your src/templates/post.js
export const pageQuery = graphql`
  query($id: String!) {
    markdownRemark(id: { eq: $id }) {
      id
      html
      frontmatter {
        title
        date(formatString: "yyyy/MM/DD")
      }
      fields {
        slug
+       encrypted
      }
    }
  }
`
```

Then you will find your `html` is encrypted, you can decrypt it use CryptoJS.

``` javascript
import CryptoJS from 'crypto-js'

if (post.fields.encrypted) {
  const realHtml = CryptoJS.AES.decrypt(post.html, '123456').toString(CryptoJS.enc.Utf8)
}
```

## Code example
Prompt user to input password.

``` javascript
// this html is encrypted
const Encrypted = ({ html }) => {
  // the decrypted content
  const [decrypted, setDecrypted] = useState('')
  // save user input
  const [password, setPassword] = useState('')
  
  return decrypted ? (
    <section
      dangerouslySetInnerHTML={{ __html: decrypted }}
      itemProp="articleBody"
    />
  ) : (
    <section itemProp="articleBody">
      <input
        type='password'
        value={password}
        onChange={e => setPassword(e.target.value)}
        onKeyUp={e => {
          if (e.keyCode === 13) {
            try {
              setDecrypted(CryptoJS.AES.decrypt(html, password).toString(CryptoJS.enc.Utf8))
            } catch (e) {
              alert('password error')
            }
          }
        }}
      />
    </section>
  )
}

// in your template, decide to show encrypted view or normal view according to `post.fields.encrypted`
export default ({ data }) => (
  ...
  {post.fields.encrypted ? (
    <Encrypted html={post.html} />
  ) : (
    <section
      dangerouslySetInnerHTML={{ __html: post.html }}
      itemProp="articleBody"
    />
  )}
  ...
)
```
