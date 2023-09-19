<script>
import { Row, Col, Collapse, Navbar, NavbarToggler, NavbarBrand, Nav, NavItem, Button, NavLink, Dropdown, DropdownToggle, DropdownMenu, DropdownItem, Form, InputGroup, Input } from 'sveltestrap'
import {site, cert, user, auth} from '../../../dir/init.js'
import MD5 from 'crypto-js/md5.js'
import { createEventDispatcher } from 'svelte'

const dispatch = createEventDispatcher()
export let isAuth
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
    // const id = MD5(useText || useFile[0]).toString()
    const stamp = Date.now()
    const sendMessage = {id: stamp + '-' + MD5(useText || useFile[0]).toString(), stamp, replied: false, replies: false, iden: isAuth, file: useFile ? useFile[0] : null, text: useText, category: document.getElementById('category').value || null, comment: document.getElementById('comment').value || null}
    const makePost = site.get('posts').get(sendMessage.id).put(sendMessage, null, {opt: {cert}})
    if(sendMessage.category){
        site.get('categories').get(sendMessage.category).get(sendMessage.id).put(makePost, null, {opt: {cert}})
    }
    site.get('users').get(sendMessage.iden).get(sendMessage.id).put(makePost, null, {opt: {cert}})
    if(sendMessage.comment){
        site.get('comments').get(sendMessage.comment).get(sendMessage.id).put(makePost, null, {opt: {cert}})
    }
    document.getElementById('file').value = null
    document.getElementById('text').value = ''
    document.getElementById('category').value = ''
    document.getElementById('comment').value = ''
    radios = null
    radio = null
    // dispatch('message', sendMessage)
    return sendMessage
}
</script>

{#if err}
    <Row class="mt-4">
        <Col>
            <p>{err}</p>
        </Col>
    </Row>
{/if}

<Row class="mt-4">
    <Col>
        <Form>
            <Input type="radio" bind:group={radios} value="bt" label="bt"/>
            <Input type="radio" bind:group={radios} value="ipfs" label="ipfs"/>
            <Input type="radio" bind:group={radios} value="hyper" label="hyper"/>
            {#if radios === 'bt'}
                <Input type="radio" bind:group={radio} value="_" label="key"/>
                <Input type="radio" bind:group={radio} value="." label="hash"/>
            {/if}
            <Input type="text" placeholder="category" id="category"/>
            <Input type="file" id="file"/>
            <Input type="textarea" placeholder="post" id="text"/>
            <Input type="text" placeholder="user" id="comment"/>
            <Button type="button" on:click={(e) => {
                handleSubmit(e).then((data) => {dispatch('message', data)}).catch((e) => {err = e.message;setTimeout(() => {err = null}, 5000);})
            }}>submit</Button>
        </Form>
    </Col>
</Row>
<style></style>