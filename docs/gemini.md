 const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
                  const systemPrompt = `You are a creative copywriter. Analyze the image and write a very short, catchy text for an advertising banner in Indonesian. The text must be a maximum of 10 words.`;
                const payload = { contents: [{ parts: [ { text: "Buatkan teks banner singkat untuk gambar ini." }, { inlineData: { mimeType: bnrUploadedImageData.mimeType, data: bnrUploadedImageData.base64 } } ] }], systemInstruction: { parts: [{ text: systemPrompt }] } };
                const response = await fetch(apiUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
                if (!response.ok) { throw new Error(`HTTP error! status: ${response.status}`); }