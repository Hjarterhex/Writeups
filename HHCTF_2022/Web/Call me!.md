# Call me!

## Challenge description

Wow! This must be the most horrible website I have ever seen. I wonder who is that desperate for a call.

https://call-me.ctfeh.hh.se

## Solution

The url leads to a web page with flashing colors and a phone number. We can assume the number has nothing to do with the challenge (the country code isn't even valid). If we inspect the source code, we can see it uses a linked javascript file, with a few functions, but only one of them seems to be called.

```javascript
const bc = ["rgb(17, 229, 45)", "rgb(232, 19, 182)"]
const t = document.getElementById("csrf-token").value
const ct = "application/x-www-form-urlencoded"
const h1 = "X-CSRF-TOKEN"
const f = "/flag"
const b = "part="
const h2 = "Content-Type"
const h = {[h1]: t, [h2]: ct}
const m = "POST"
const np = parseInt(document.getElementById("np").value)
var c = 0

function cc() {
    if (c % 5 == 0) {
        document.body.style.backgroundColor = bc[c % 2]
        document.getElementById("call-me").style.color = bc[(c + 1) % 2]
        document.getElementById("await").style.color = bc[(c + 1) % 2]
    }
    for (let i = 1; i <= 17; i++) {
        let e = document.getElementById(i.toString())
        let n = parseInt(e.className.slice(1))
        console.log()
        e.className = "c" + ((n + 1) % 12)
    }
    c++
}

async function me() {
    if (document.visibilityState == "hidden") {
        return false
    }
    let p = await gf()
    let f = []
    for (let i = 0; i < np; i++) {
        f[p[np][i]] = p[i]
    }
    return f.join('')
}

async function gf() {
    let p = []
    for (let i = 0; i < np; i++) {
        p[i] = await gp(i)
    }
    p[np] = await gl()

    return p
}

async function gp(p) {
    return await fetch(f, {method: m, headers: h, body: b + p})
    .then((r) => { return (r.ok && r.text()) })
}

async function gl() {
    return await fetch(f, {method: m, headers: h, body: b + np})
    .then((r) => { return (r.ok && r.json()) })
}

setInterval(cc, 125)
```
The function and variable names is a bit cryptic, but we can see that the `me()` function calls the `gf()` function, which in turn calls the `gp()`and `gl()` functions. These functions uses fetch, to get some data from `/flag`, using `POST`-requests. We could go for mimicking the requests, but why bother, since it seems to already be done here? Instead we go to the console and call the `me()` function (seems pretty obvious when we think about it). since it's an `async` function we use the `await` keyword (another hint from the page).

Note: the

```javascript
    if (document.visibilityState == "hidden") {
        return false
    }
```
part of the code means we have to have the page visible, or the function will fail. (We could of course remove that part in the sources tab if we want to)

![Get the flag](../../../../../D:/ctf/Writeups/HHCTF_2022/img/call_me_1.png)

Flag: `HHCTF{1'm_n3v3r_m0r3_th4n_4_func710n_c4ll_4w4y}`