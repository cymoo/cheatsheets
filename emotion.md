# Emotion

1. 配置

   1. Babel preset：[emotion.sh](https://emotion.sh/docs/css-prop#babel-preset)
   2. JSX Pragma：文件顶部 /** @jsxImportSource @emotion/react */

2. String Styles

   ```jsx
   <div css={css` background-color: hotpink; &:hover { color: ${color}; } `} >
   ```

3. Object Styles

   ```jsx
   <div css={{ backgroundColor: 'hotpink', '&:hover': { color: 'lightgreen' } }} >
   ```

4. 嵌套

   ```jsx
   css` color: turquoise; a { cursor: pointer; } `
   
   css` color: turquoise; header & { color: green; } `
   ```

5. media query

   ```jsx
   <p css={css` font-size: 30px; @media (min-width: 420px) { font-size: 50px; } `} >
   ```

6. Reusable media queries: [emotion.sh](http://emotion.sh/) [emotion.sh](http://emotion.sh/)

   ```jsx
   const breakpoints = [576, 768, 992, 1200]
   const mq = breakpoints.map(bp => @media (min-width: ${bp}px)) <div
     css={{
       color: 'green',
       [mq[0]]: {color: 'grey'},
       [mq[1]]: {color: 'red'},
     }}
   >
   ```

7. Global styles

   ```jsx
   import { Global, css } from '@emotion/react'
   render(
     <div>
       <Global
         styles={css`
           .some-class {
             color: hotpink !important;
         }`}
       />
       <Global
         styles={{
         '.some-class': {
           fontSize: 50,
           textAlign: 'center'
         }}}
       />
       <div className="some-class">This is hotpink now!</div> </div>)
   ```

8. 组合（重要）

   1. 字符串插值

      ```jsx
      const style1 = css`
        border: 3px solid yellow;
        a { color: #ffa7c4; }
      `
      
      const style2 = css`
        background-color: #8ddeb5;
        ${style1};
      `
      ```

   2. 数组

      ```jsx
      <p className={className} css={[ css` font-size: 24px; `, style2, ]} >
      ```

9. Wrapper

   1. 以下无效

      ```jsx
      const Wrapper = ({ wrapperClassName, className }) => { return (
        <div className={wrapperClassName}>
          <p className={className}>hello</p>
        </div>
      )}
      
      <Wrapper
        wrapperClassName={css` background-color: #55bb8e; }
        css={css color: red; `}
      />
      ```

   2. 需要使用 ClassNames 的render prop

      ```jsx
      import { ClassNames } from '@emotion/react'
      // this might be a component from npm that accepts a wrapperClassName prop
      
      let SomeComponent = props => (
        <div className={props.wrapperClassName}>
          in the wrapper!
          <div className={props.className}>{props.children}
          </div>
        </div>
      )
      
      <ClassNames>
        render({({ css, cx }) => (
          <SomeComponent
            wrapperClassName={css({ color: 'green' })}
            className={css` color: hotpink; `} > from children!!
          </SomeComponent> )}
      </ClassNames> )
      ```

10. Keyframes

    ```jsx
    import { css, keyframes } from '@emotion/react'
    
    const bounce = keyframes`
      from, 20%, 53%, 80%, to {
        transform: translate3d(0,0,0);
      }
    
      40%, 43% {
        transform: translate3d(0, -30px, 0);
      }
    
      70% {
        transform: translate3d(0, -15px, 0);
      }
    
      90% {
        transform: translate3d(0,-4px,0);
      }
    `
    
    render(
      <div
        css={css`
          animation: ${bounce} 1s ease infinite;
        `}
      >
        some bouncing text!
      </div>
    )
    ```

11. Theme: [emotion.sh](http://emotion.sh/)

    ```jsx
    import { ThemeProvider, useTheme } from '@emotion/react'
    
    const theme = {
      colors: {
        primary: 'hotpink'
      }
    }
    
    function SomeText(props) {
      const theme = useTheme()
      return <div css={{ color: theme.colors.primary }} {...props} />
    }
    
    render(
      <ThemeProvider theme={theme}>
        <SomeText>some text</SomeText>
      </ThemeProvider>
    )
    ```

12. 性能优化

    1. React.memo等常规方式
    2. 静态样式用css prop，动态用style prop
    3. 将css``放在组件外面，避免每次render时都执行
    4. 将css prop放在父组件中，避免在大量循环中用css prop