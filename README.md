const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs');
const { logApiCall } = require('../logger');
const router = express.Router();
const upload = multer({ dest: path.join(__dirname, '..', 'uploads/') });

/**
 * POST /api/stt/file
 * Accepts audio file (max file size check should be done by front-end and multer)
 * Returns transcription text.
 */
router.post('/file', upload.single('audio'), async (req, res) => {
  const start = Date.now();
  try {
    if (!req.file) return res.status(400).json({ error: 'No audio file provided' });

    const filePath = req.file.path;
    // TODO: Send file to Whisper / OpenAI STT here and get transcription
    // Example pseudocode:
    // const transcription = await openai.audio.transcriptions.create({ file: fs.createReadStream(filePath), model: 'whisper-1' });

    // For scaffold we return a placeholder and instruct how to plug actual call:
    const transcription = { text: '<<TRANSCRIPTION_PLACEHOLDER - integrate OpenAI Whisper or other STT>>' };

    // Log to DB
    await logApiCall({
      api_name: 'STT',
      latency_ms: Date.now() - start,
      tokens_used: null,
      cost_estimate: null,
      meta: { filename: req.file.originalname }
    });

    // cleanup uploaded file
    fs.unlink(filePath, () => {});

    return res.json({ transcription: transcription.text });
  } catch (err) {
    console.error('STT error', err);
    return res.status(500).json({ error: 'Error processing STT' });
  }
});

module.exports = router;
