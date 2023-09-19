<script>
import { Row, Col, Collapse, Navbar, NavbarToggler, NavbarBrand, Nav, NavItem, Button, NavLink, Dropdown, DropdownToggle, DropdownMenu, DropdownItem, Form, InputGroup, Input } from 'sveltestrap'
import {site, cert, user} from '../../../dir/init.js'
import MD5 from 'crypto-js/md5.js'
import { createEventDispatcher } from 'svelte'

const dispatch = createEventDispatcher()
export let isAuth
export let post
let radios
let radio
let err
function handleType(){
    if(radios === 'bt'){
        return radio
    } else if(radios === 'ipfs'){
        return '.'
    } else if(radios === 'hyper'){
        return '_'
    } else {
        throw new Error('type of post is not valid')
    }
}

async function handleSubmit(e){
    console.log(e)
    if((!document.getElementById('file').files || !document.getElementById('file').files.length) && !document.getElementById('text').value){
        return
    }
    if((document.getElementById('file').files && document.getElementById('file').files.length) && ((radios === 'bt' && !radio) || !radios)){
        return
    }
    let useFile
    if(document.getElementById('file').files && document.getElementById('file').files.length){
        const formData = new FormData()
        for(let i = 0;i < document.getElementById('file').files.length;i++){
            formData.append(document.getElementById('file').files[i].name, document.getElementById('file').files[i])
        }
        useFile = JSON.parse(await (await fetch(`${radios}://${handleType()}/`, {method: 'POST', headers: {'Accept': 'application/json'}, body: formData})).text())
    }
    let useText = document.getElementById('text').value
    const stamp = Date.now()
    const sendMessage = {id: stamp + '-' + MD5(useText || useFile[0]).toString(), replied: true, replies: false, stamp, iden: isAuth, file: useFile ? useFile[0] : null, text: useText}
    const makePost = site.get('posts').get(sendMessage.id).put(sendMessage, null, {opt: {cert}})
    const getPost = site.get('posts').get(post.id)
    makePost.get('to').get(post.id).put(getPost, null, {opt: {cert}})
    getPost.get('from').get(sendMessage.id).put(makePost, null, {opt: {cert}})
    if(!post.replies){
        getPost.get('replies').put(true, null, {opt: {cert}})
    }
    // site.get('categories').get('category-' + sendMessage.category).get(sendMessage.id).put(makePost, null, {opt: {cert}})
    // site.get('users').get('user-' + sendMessage.iden).get(sendMessage.id).put(sendMessage, null, {opt: {cert}})
    document.getElementById('file').value = null
    document.getElementById('text').value = ''
    radios = null
    radio = null
    // dispatch('message', sendMessage)
    return sendMessage
}
</script>

{#if err}
    <Row>
        <Col>
            <p>{err}</p>
        </Col>
    </Row>
{/if}
<Row>
    <Col>
        <Form>
            <Input type="radio" bind:group={radios} value="bt" label="bt"/>
            <Input type="radio" bind:group={radios} value="ipfs" label="ipfs"/>
            <Input type="radio" bind:group={radios} value="hyper" label="hyper"/>
            {#if radios === 'bt'}
                <Input type="radio" bind:group={radio} value="_" label="key"/>
                <Input type="radio" bind:group={radio} value="." label="hash"/>
            {/if}
            <Input type="file" id="file"/>
            <Input type="textarea" placeholder="post" id="text"/>
            <Button type="button" on:click={(e) => {
                handleSubmit(e).then((data) => {dispatch('message', data)}).catch((e) => {err = e.message;setTimeout(() => {err = null}, 5000);})
            }}>submit</Button>
        </Form>
    </Col>
</Row>

<style></style>