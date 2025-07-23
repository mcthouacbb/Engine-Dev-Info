The most basic form of an NNUE network consists of three layers: an input layer of length 768, one hidden layer of arbitrary size, and an output layer consisting of one neuron, representing the evaluation of the position. A NNUE network also commonly consists of two perspectives. That is, two hidden layers representing both sides are concatenated into a single hidden layer of twice the length, before being forwarded to the output layer.

### Input Layer

The input layer of a basic NNUE network contains 768 neurons of binary values, each representing one possible combination of piece, square, and color. Each neuron can be either on or off, depending on whether a piece with the corresponding combination exists on the board. The following is a common encoding scheme seen in most NNUE engines.

### Accumulator

The first hidden layer of a NNUE is also called the accumulator. The accumulator can be of arbitrary size, but integer multiples of powers of 2 are preferred as they allow the inference to be optimized more effectivly. A large hidden layer trades speed for higher evaluation accuracy, and vice versa. Larger hidden layers are usually preferred as Elo gain from speedup tends to scale inversely with respect to time and computational power. Most top engines have accumulator sizes of more than 1024 neurons, with some even using accumulators as wide as 3072. A large accumulator is important as it is the only hidden layer that can be efficiently updated.

### Perspective

The vast majority of NNUE architectures require two accumulators to be maintained separately. Each represent the board from the "perspective", or point-of-view, from each side. The current board position is always from the white perspective. To calculate the input values from the perspective of black, the board is flipped vertically. This means that piece's color is inverted, and its square is mirrored across the center of the board (e.g., A1 becomes A8). Both perspectives use identical input weights and biases. During inference, the two accumulators are concatenated, with the accumulator representing side-to-move being first. This resulting vector of double the accumulator width is the "true" hidden layer, and is the vector that gets forwarded to the output layer.

### Efficient Updates

The format of the input layer allows the accumulators to be efficiently updated, meaning that the neural network does not need to be fully re-evaluated after making a move. Quiet moves and promotions can only change the values of two neurons in the input layers, while captures and castling can change the values of three and four input neurons, respectively. Therefore, instead of fully calculating the accumulator values, one can take the values of the accumulator from the previous ply, subtract the weight contributions from neurons that will be turned off, and add the weights from neurons that will be turned on. Due to the nonlinearity introduced by the activation function, these efficient updates can only be performed on the first hidden layer of the neural network.

### Output Layer

Most NNUE networks have exactly one output neuron, indicating the heuristic score of a position. The accumulator values are passed through an activation function, then forwarded to the output layer by taking the dot product with the output weights, and adding the output bias.

During training, the final evaluation is passed through sigmoid activation to be trained on a target value. The target value is the result of a linear interpolation of the game result (0, 0.5, 1), and `sigmoid(evaluation/eval_scale)`.

However, during inference, the sigmoid function is unused, and the final evaluation is scaled by `eval_scale`. This will produce a reasonable evaluation with a large range and sufficient precision.  

```c
// When forwarding the accumulator values, the network does not consider the color of the perspectives.
// Rather, we are more interested in whether the accumulator is from the perspective of the side-to-move.
int32_t forward(const struct Network*     const network,
                const struct Accumulator* const stm_accumulator,
                const struct Accumulator* const nstm_accumulator)
{
    int32_t eval = 0;

    // Dot product to the weights
    for (int i = 0; i < HL_SIZE; i++)
    {
        // BEWARE of integer overflows here.
        eval += activation( stm_accumulator->values[i]) * network->output_weights[i];
        eval += activation(nstm_accumulator->values[i]) * network->output_weights[i + HL_SIZE];
    }

    // Uncomment the following dequantization step when using SCReLU
    // eval /= QA;
    eval += network->output_bias;

    eval *= SCALE;
    eval /= QA * QB;

    return eval;
}
```

### Quantization

You may be wondering where the constants QA and QB came from, or why the final evaluation is divided by their product. This is because most NNUE networks are quantized. During training, the weights of the network are represented in floating points, since they offer a continuous range of values and reasonable accuracy. However, floating points are slow, and since floating point operations are not completely accurate, small errors could build up on each accumulator update, resulting in inaccurate evaluation. Therefore, trained floating-point weights are quantized before loading the network.

Quantization of a NNUE proceeds as follows: multiply weights and biases by some amount, then round the values into an integer. This will lose some precision, but not enough to significantly affect evaluation accuracy. Suppose we quantize the input-accumulator weights by QA, then one floating point at the accumulator will correspond to QA after quantization. Therefore, the accumulator biases are also scaled by QA, and the activation function no longer clips the value to 1, but to QA. Suppose we then quantize the accumulator-output weights by QB, then, assuming CReLU activation, one floating point value at the output node will correspond to QA * QB after quantization. Thus, the output bias is scaled by QA * QB. At the end of inference, we divide out the quantization, leaving us a very close approximation of the network evaluation.

SCReLU activation makes things messier. A floating point value of the accumulator values after SCReLU is actually QA * QA, so the output bias is scaled by QA * QA * QB. However, this does not fit inside a 16-bit integer. To prevent this, some trainers will still scale the output bias by QA * QB, so you would divide the dot product by QA preliminarily. This puts the output scaling back to QA * QB, and the dequantization can continue as usual.
