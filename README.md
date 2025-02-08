<div align="center">

# R1-Vision: Let's first take a look at the image

<div align="center">
  <img width="640" alt="image" src="https://github.com/user-attachments/assets/f917bf13-674a-4d17-a6ea-8e141addb3fc">
  <br>
</div>

[\[🤗 Cold-Start Dataset\]](https://huggingface.co/collections/yuyq96/r1-vision-67a6fb7898423dca453efa83)  [\[📜 Report (Coming Soon)\]]()

</div>

DeepSeek-R1 demonstrates outstanding reasoning abilities when tackling **math**, **coding**, **puzzle**, and **science** problems, as well as responding to **general** inquiries. However, as a text-only reasoning model, R1 cannot process multimodal inputs like images, which limits its practicality in certain situations. Exploring the potential for multimodal reasoning is an intriguing prospect.

We build this project to create a model that can reason with both text and images.

## 🔥 News

- `2025/02/08`: We are excited to announce the release of the initial version of our cold-start dataset on [🤗 HuggingFace](https://huggingface.co/collections/yuyq96/r1-vision-67a6fb7898423dca453efa83). This first trial employs the `DeepSeek-R1-Distill-Qwen-32B` model for reasoning and the `GPT-4o-mini` model for image captioning and data formatting. 

> [!NOTE]
> We are actively working on developing enhanced versions that will:
> - Incorporate more powerful models.
> - Increase task diversity.
> - Improve sample quality.
> 
> Stay tuned for updates!

## 🚀 Stage1: Cold Start

We explore distilling the strong reasoning capabilities from a Large Language Model (LLM) such as R1 to a Large Vision-Language Model (LVLM). Specifically, we utilize three kinds of data, including:

- **Text Data**: Text-only reasoning datasets.
- **Text Rendering Data**: Curated from text-only reasoning datasets, ultilizing a reformatting and rendering pipeline. We adopt these data to encourage identical response to different modality inputs.
- **Multimodal Data**: Curated from raw multimodal datasets. We adopt a simple strategy to mitigate the absence of vision capabilities in text-only reasoning models, called **Caption-Prefixing**.

| Type | Source Dataset| Numbers |
| :- | :- | :-: |
| Text | Bespoke-Stratos-17k | 16.7k |
| Text Rendering | Bespoke-Stratos-17k | 12.6k |
| Multimodal | AI2D | 7.8k |
| Text / Multimodal | ScienceQA | 9.9k |
| Multimodal | PixMo-Cap-QA | 19.4k |

### 👁 Caption-Prefixing

Similar to the reasoning forcing trick, we make the model pretend to "see" the image by captioning the image in the beginning of the thinking process. We use a simple template:

```python
# English
prefix_en = "<think>\nHmm, to solve this problem, let's first take a look at the image. {}\n\nNow".format(image_caption)

# Chinese
prefix_zh = "<think>\n嗯，为了回答这个问题，让我先看一下图片。{}\n\n首先".format(image_caption)
```

There are two important **switches** for entering and exiting the **caption mode**:
- Enter: `let's first take a look at the image.`
- Exit: `Now`

> [!TIP]
> It is worth noting that the exit switch originates from the original R1 thought process, which helps the model stop captioning and avoids hallucinations.

This method achieved well-formatted thoughts and solutions, without the need for heavy post-processings like LLM reformatting. To clarify, we can diversify the switch styles by string replacement.

#### Examples

| Image | Caption | Thought | Solution |
| - | - | - | - |
| ![c8e3f7c11306f3dd1cf3d73118963eab](https://github.com/user-attachments/assets/32fe916b-2f08-4c8a-a2ef-1af29fc7c92e) | The image features a close-up view of the legs of a frog, prominently displaying its unique anatomical characteristics. The frog is positioned on a wooden surface, which introduces a warm brown hue that contrasts with the pale yellow of the frog's legs.<br>... | Now, considering the question: which animal's feet are also adapted for sticking to smooth surfaces? The options are meerkat and Madagascar day gecko.<br>Given that the image shows a frog, which uses sticky pads on its feet to adhere to surfaces, the most likely answer is the Madagascar day gecko, as geckos are known for their ability to climb smooth surfaces using specialized pads. Meerkats, on the other hand, are more adapted for digging and do not have such sticky feet. | The correct answer is:<br>**B. Madagascar day gecko**<br>Geckos, including the Madagascar day gecko, have specialized toe pads that allow them to stick to smooth surfaces. This adaptation is due to millions of tiny hairs called setae, which create a strong adhesive force through molecular attraction. |
| ![51a35e884e8aac91c5419251fe27dbb9](https://github.com/user-attachments/assets/6cf30ed5-670e-4423-a335-fe6314875a91) | The image depicts a stylized representation of a scarf that features a bold pattern of alternating stripes. The scarf predominantly displays red and cream-colored stripes, with the red stripes being more prominent. The design consists of several equal-width horizontal stripes: the upper part of the scarf shows a vibrant red stripe followed by a creamy ivory stripe, and this pattern continues down the length of the scarf.<br>... | Now, considering the question: Is a scarf a solid or a liquid? The options are A. a solid and B. a liquid. From the description and the image, it's clear that a scarf is a physical object made from materials like wool or yarn. These materials are solid in their form. Liquids, on the other hand, take the shape of their container and flow freely, which does not apply to a scarf. Therefore, the scarf is a solid. | A. a solid |

### 📝 Text Rendering

- Reformatting the original question with an LLM.
- Rendering the reformatted LaTeX files on images.

#### Examples

| Original Text | Rendered Image |
| - | - |
| <span>$</span>P(x)<span>$</span> is a polynomial of degree <span>$</span>3n<span>$</span> such that\n\\begin{eqnarray*} P(0) = P(3) = \\cdots &=& P(3n) = 2, \\\\ P(1) = P(4) = \\cdots &=& P(3n-2) = 1, \\\\ P(2) = P(5) = \\cdots &=& P(3n-1) = 0, \\quad\\text{ and }\\\\ && P(3n+1) = 730.\\end{eqnarray*}\nDetermine $n$. | ![2](https://github.com/user-attachments/assets/aaa6ab02-9de6-4a79-a82c-d7cc2f8303f6) |
| Jose, Thuy, and Kareem each start with the number 10.  Jose subtracts 1 from the number 10, doubles his answer, and then adds 2.  Thuy doubles the number 10, subtracts 1 from her answer, and then adds 2.  Kareem subtracts 1 from the number 10, adds 2 to his number, and then doubles the result.  Who gets the largest final answer? | ![8](https://github.com/user-attachments/assets/94548483-9208-4204-baea-6872b8cb5505) |

### 📈 Performance

TODO: Train and evaluate TextHawk2-7B and Qwen2.5-VL-7B.

## 🧠 Stage2: RL

TODO: Explore RL for LVLMs.

## 👨🏻‍💻 Citation

If you find this project useful in your research, please consider cite:

```
@misc{yu25r1vision,
  author       = {Ya{-}Qi Yu and Minghui Liao and Jihao Wu and Chao Weng},
  title        = {R1-Vision: Let's first take a look at the image},
  howpublished = {\url{https://github.com/yuyq96/R1-Vision}},
  note         = {Accessed: 2025-02-08},
  year         = {2025}
}
```

## 🤝 Acknowledgement

R1-Vision is built with reference to the code or data of the following projects: [DeepSeek-R1](https://github.com/deepseek-ai/DeepSeek-R1), [Bespoke-Stratos-17k](https://huggingface.co/datasets/bespokelabs/Bespoke-Stratos-17k), [AI2D](https://prior.allenai.org/projects/diagram-understanding), [ScienceQA](https://scienceqa.github.io/), [PixMo](https://huggingface.co/collections/allenai/pixmo-674746ea613028006285687b). Thanks for their awesome work!
