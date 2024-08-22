const express = require('express');
const twilio = require('twilio');
const axios = require('axios');
require('dotenv').config();

const app = express();
const port = process.env.PORT || 3000;

app.use(express.urlencoded({ extended: false }));
app.use(express.json());

// Função para buscar música no Spotify
async function searchSpotify(query) {
    try {
        const token = await getSpotifyToken();
        const result = await axios.get(`https://api.spotify.com/v1/search?q=${query}&type=track`, {
            headers: {
                Authorization: `Bearer ${token}`
            }
        });
        return result.data.tracks.items[0];
    } catch (error) {
        console.error("Erro ao buscar música:", error);
    }
}

// Função para obter o token de acesso ao Spotify
async function getSpotifyToken() {
    const authOptions = {
        method: 'post',
        url: 'https://accounts.spotify.com/api/token',
        headers: {
            'Authorization': 'Basic ' + (Buffer.from(process.env.SPOTIFY_CLIENT_ID + ':' + process.env.SPOTIFY_CLIENT_SECRET).toString('base64')),
            'Content-Type': 'application/x-www-form-urlencoded'
        },
        data: 'grant_type=client_credentials'
    };
    const response = await axios(authOptions);
    return response.data.access_token;
}

// Webhook do Twilio para receber mensagens
app.post('/whatsapp', async (req, res) => {
    const twiml = new twilio.twiml.MessagingResponse();
    const incomingMsg = req.body.Body.trim();

    const music = await searchSpotify(incomingMsg);

    if (music) {
        twiml.message(`Achei uma música! ${music.name} - ${music.artists[0].name}\nLink: ${music.external_urls.spotify}`);
    } else {
        twiml.message('Desculpe, não consegui encontrar essa música.');
    }

    res.writeHead(200, { 'Content-Type': 'text/xml' });
    res.end(twiml.toString());
});

app.listen(port, () => {
    console.log(`Bot rodando na porta ${port}`);
});TWILIO_ACCOUNT_SID=your_twilio_account_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token
SPOTIFY_CLIENT_ID=your_spotify_client_id
SPOTIFY_CLIENT_SECRET=your_spotify_client_secrethttps://accounts.spotify.com/api/tokenhttps://api.spotify.com/v1/search?q=${query}&type=track
    console.log(`Bot rodando na porta ${port}`);
})music.nameindex.jsnode index.js
